# Resilience-паттерны: Timeouts, Retries, Circuit Breaker, Bulkhead, Rate Limit, Fallback


| Взаимодействие                             | Таймаут | Retries                    | CB  | Bulkhead                | Rate Limit | Fallback  | Обоснование                                                                                                                                                          |
|--------------------------------------------|---------|----------------------------|-----|-------------------------|------------|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| BFF → Wallet Service                       | 1s      | нет                        | нет | Thread pool             | да         | нет       | Синхронный запрос от клиента, ожидается быстрый ответ. Rate limit защищает от клиентских всплесков и DDoS                                                            |
| Wallet → Transaction Service               | —       | Kafka retry + DLQ          | —   | Consumer group          | —          | DLQ       | Асинхронное взаимодействие через Kafka, at-least-once доставка, ошибки обрабатываются через DLQ                                                                      |
| Transaction → Payment Service              | —       | Kafka retry + DLQ          | —   | Consumer group          | —          | DLQ       | Асинхронная передача команд, устойчивость за счёт брокера сообщений                                                                                                  |
| Payment Service → Payment Provider         | 2s      | 3 (exp. backoff + jitter)  | да  | Отдельный   client pool | да         | PENDING   | Только в случае, если Payment Provider имеет защиту от двойных списаний!!! Внешний нестабильный сервис, нужен CB, retries с backoff, fallback в отложенную обработку |
| BFF → Payment Query Service                | 1s      | нет                        | нет | Thread pool             | да         | нет       | Read-only операции, предпочтительнее быстрый отказ                                                                                                                   |
| Notification → Notification Provider       | 3s      | 5 (exp. backoff + jitter)) | да  | Отдельный client pool   | да         | DLQ       | Внешний провайдер, допустима задержка доставки                                                                                                                       |
| API Gateway → Wallet/Payment Query Service | 1s      | нет                        | да  | Connection pool         | да         | 429 / 503 | Первая линия защиты: лимиты, таймауты, защита от каскадных отказов                                                                                                   |

# Payment Service → Payment Provider

Ретраим только технические ошибки:
- timeout;
- connection reset;
- HTTP 5xx;
- HTTP 429.

Параметры:
- maxAttempts = 3
- backoff = exponential
- initialDelay = 200ms
- maxDelay = 2s
- jitter = ±30%

Circuit Breaker:
- Sliding window: 50 последних вызовов
- Failure rate threshold: ≥ 50%
- Slow call threshold: ≥ 2s
- Slow call rate threshold: ≥ 50%
- Open state duration: 30 секунд

Half-Open:
- разрешаем 5 пробных запросов;
- при ≥1 ошибке — возвращаемся в OPEN;
- при 100% успехе — CLOSED.

Fallback:
Если все retry исчерпаны или Circuit Breaker в состоянии OPEN, тогда:
- платёж переводится в PENDING_PROVIDER
- событие отправляется в Kafka;
- клиент получает 202 Accepted.

# Notification Service → Notification Provider

Ретраим только технические ошибки:
- timeout;
- HTTP 5xx.

Параметры:
- maxAttempts = 5
- backoff = exponential + jitter

Circuit Breaker:
- Быстро уходим в OPEN, чтобы не блокировать основной поток.

Fallback:
- Сообщение отправляется в DLQ;
- Возможна ручная или отложенная повторная отправка.

---

# Очереди, DLQ и backpressure

## Основные топики

| Топик              | Назначение                   | Producer            | Consumer                                   |
|--------------------|------------------------------|---------------------|--------------------------------------------|
| payments.initiated | Инициация платежа            | Wallet Service      | Transaction Service, Payment Query Service |
| payments.callback  | Получение финального статуса | Callback Service    | Transaction Service                        |
| payments.result    | Результат обработки платежа  | Transaction Service | Wallet Service, Payment Query Service      |
| payments.completed | Отправка уведомлений         | Wallet Service      | Notification Service                       |

---

## Dead Letter Queue (DLQ)


| Cценарий                                                     | Тип сообщения | Producer            | Consumer                                   | Основной топик     | DLQ                        | Комментарий по DLQ-обработке                                                                                                                                                                                                                        |
|--------------------------------------------------------------|---------------|---------------------|--------------------------------------------|--------------------|----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Wallet Service → Transaction Service / Payment Query Service | command       | Wallet Service      | Transaction Service, Payment Query Service | payments.initiated | payments.wallet.dlq        | Сообщения попадают в DLQ при ошибке маппинга или неконсистентных данных. Автоматический retry ограничен, так как повторная инициация платежа может привести к дубликатам. Разбор — ручной с возможной повторной отправкой после исправления данных. |
| Callback Service → Transaction Service                       | event         | Callback Service    | Transaction Service                        | payments.callback  | provider.callback.dlq      | Используется при невалидном payload или нарушении контракта провайдера. Повторная обработка возможна после исправления формата или логики парсинга.                                                                                                 |
| Transaction Service → Wallet Service / Payment Query Service | event         | Transaction Service | Wallet Service, Payment Query Service      | payments.result    | payments.transactional.dlq | Сообщения попадают в DLQ при ошибке бизнес-валидации или несоответствии статуса. Автоматический retry отключён, так как проблема чаще логическая. Требуется анализ и ручное решение.                                                                |
| Wallet Service → Notification Service                        | event         | Wallet Service      | Notification Service                       | payments.completed | payments.notifications.dlq | Ошибки доставки. События идемпотентны, допускается повторная обработка без влияния на финансовый контур.                                                                                                                                            |

DLQ мониторятся по:
- количеству сообщений;
- времени нахождения сообщения в DLQ;

---

## Backpressure и обработка всплесков нагрузки

Ограничение конкуренции консьюмеров:
Transaction Service:
- 4 consumers threads на инстанс (CPU-bound бизнес-логика и операции с БД, масштабирование по lag в payments.initiated и payments.callback)
Wallet Service:
- 2 consumers threads на инстанс (операции с балансами и требования к порядку обработки)
Notification Service:
- 8 consumers threads на инстанс (отсутствие влияния на финансовый контур)

Очереди внутри обработчиков:
- фиксированный размер очереди
- при заполнении:
    - consumer перестаёт забирать новые сообщения
    - offset не коммитится
    - сообщения остаются в брокере и не теряются
    - замедление обработки
    - нефинансовые операции (уведомления, логирование) не блокируют критический путь платежа


Поведение при перегрузке:
- При достижении пороговых значений применяется следующий набор мер:
    - временная приостановка чтения из брокера
    - backpressure поднимается на уровень Kafka

Стратегия retry без retry-storm:
- retry выполняется с экспоненциальной задержкой. 
- retry реализован через отдельные retry-topics с увеличивающейся задержкой
- количество попыток ограничено


---

# Кэширование (Cache-Aside / Read-Through / Write-Through)

Используем Redis DB в качестве кэша для получения информации о платежах.

## Обоснование выбора

Cache-Aside выбран:
- легко контролировать консистентность;
- не влияет на write-потоки;
- только "нужные" данные будут в кэше.

Денежные операции не используют кэш:
- баланс и резервы — только из write-модели;
- все изменения через транзакции и outbox.
- Eventual consistency допустима только для чтения:
- пользователь может увидеть статус с небольшой задержкой;
- деньги при этом всегда корректны.

## Защита от cache stampede (массовых запросов)
- Только один запрос читает из ClickHouse при cache miss (остальные ждут результат из Redis).
- Популярные запросы (дашборды, отчёты) периодически прогреваются.

## Обновление данных
При PaymentResult: 
- Полная перезапись записи в Redis
- Статус: PENDING_PROVIDER (TTL 60s).
- Статусы: SUCCESS, FAILED (TTL 24h).

При PaymentInitiated:
- Установка короткого TTL (30s)
- Данные также пишутся в ClickHouse.

Архивные данные (история операций за дни/месяцы) не кэшируем - берем с ClickHouse.

---

# Observability: метрики, логи, трейсы, SLI/SLO

## Стек

Используем Prometheus + Grafana (метрики), Loki (логи), OpenTelemetry + Grafana Tempo(трейсы) для сбора метрик, логов и трейсов.

Prometheus:
- Стандарт для Kubernetes и микросервисов

Grafana:
- Универсальный UI для метрик, логов и трейсов.
- Поддержка SLO-дашбордов.
- Возможность быстро показать метрики

Loki:
- Логи индексируются по labels.
- Отлично работает в Kubernetes.

OpenTelemetry:
- Стандарт

Tempo:
- Глубокая интеграция с Grafana.

---

## Метрики

API Gateway:
- количество успешных/неуспешных запросов
- max время ответа
- количество ответов > 5s

Wallet:
- количество успешных/неуспешных платежей

Transactional:
- количество успешных/неуспешных платежей

Callback:
- количество успешных/неуспешных платежей

Query:
- количество успешных/неуспешных отчетов
- количество успешных/неуспешных чеков об оплате
- количество успешных/неуспешных платежей

Notification Service:
- retry
- circuit breaker metrics

Payment Service:
- retry
- circuit breaker metrics

Payment Provider:
- latency
- количество ошибок (500/502/503)

Notification Provider:
- latency
- количество ошибок (500/502/503)

Kafka
- размер очередей
- лаг consumer-а

RabbitMQ:
- размер очередей
- лаг consumer-а

---

# SLI/SLO

| SLI                                | SLO                | Где измеряется                       |
|------------------------------------|--------------------|--------------------------------------|
| Успешность createPayment (2xx/4xx) | ≥ 99.9% за 30 дней | API Gateway                          |
| p95 latency createPayment          | ≤ 2 секунды        | API Gateway + Wallet Service         |
| Доля платежей в INCONSISTENT       | 0%                 | Wallet Service + Transaction Service |
| Успешность провайдера              | ≥ 99%              | Payment Service                      |
| Lag consumer `payments.result`     | ≤ 5 секунд         | Kafka (consumer lag metrics)         |

---

## Дашборды

Operational dashboard по платежам.
- График с количеством успешных/неуспешных платежей за последние 5 дней. 
- Таблица с неуспешными платежами и ошибками по ним.
- График с количеством неуспешных обращений к Payment Provider за последние 5 дней. 

Notification dashboard.
- График с количеством неуспешных обращений к Notification Provider за последние 5 дней. 
- Количество успешных/неуспешных уведомлений за последние 5 дней.
- Таблица с неуспешными уведомлениями и ошибками по ним.

---

## Масштабирование и изоляция

# Wallet Service
- Масштабируется горизонтально по RPS.
- При росте нагрузки масштабируется число pod-ов. Можно рассмотреть масштабирование БД (read replicas).

# Transaction Service
- Масштабируется горизонтально как Kafka consumer group.
- Количество инстансов ≤ количество партиций топика.
- Каждый paymentId обрабатывается строго одним consumer-ом.

# Payment Query Service
- Масштабируется горизонтально по RPS.
- Использует:
    - Redis cache (cache-aside)
    - аналитическую БД (ClickHouse)

# Notification Service
- Масштабирование зависит от возможностей Notification Provider.
- Масштабируется горизонтально как Kafka consumer group.
- Количество инстансов ≤ количество партиций топика.

# Callback Service
- Масштабируется горизонтально для приёма callback-ов от провайдера.
- Stateless, устойчив к повторным callback-ам за счёт идемпотентности.

# Payment Service
- Масштабирование зависит от возможностей Payment Provider.
- Масштабируется горизонтально как Kafka consumer group.
- Количество инстансов ≤ количество партиций топика.

# Kafka
Масштабирование достигается за счёт:
- Увеличения количества партиций в топиках.
- Добавления consumer-ов в consumer group.

# RabbitMQ
- Масштабируется за счет увеличения числа консьюмеров.

---

# Разделение потоков и изоляция нагрузки

- Один топик — один тип события.
- Consumer Groups:
    - Wallet, Transaction, Query, Notification — независимые группы;
    - сбой одного consumer-а не влияет на других.
- Bulkhead-изоляция:
    - отдельные thread pools:
        - для Kafka consumer-ов;
        - для HTTP вызовов провайдера;
        - для работы с БД.

Ключи лимитирования:

| Сервис               | Ключ       | Ответ |
|----------------------|------------|-------|
| API Gateway          | clientId   | 429   |
| Payment Service      | providerId | 503   |
| Notification Service | providerId | 503   |

Burst-поведение

Допускаем short burst:
- Token Bucket;
- ограниченный размер burst-а.

Что изолируем:

Payment Service:
- providerHttpPool — только для вызовов Payment Provider
- kafkaConsumerPool — только для Kafka сообщений
- dbPool — только для транзакций БД

Wallet Service:
- apiRequestPool — входящие HTTP запросы
- eventConsumerPool — обработка Kafka событий
- dbPool — операции с балансами и резервами

---

### Ожидаемый эффект от Bulkhead-изоляции

| Риск                | Как предотвращается                                                                |
|---------------------|------------------------------------------------------------------------------------|
| Каскадный отказ     | Проблема в одном пуле (HTTP к провайдеру) не влияет на другие пулы                 |
| Starvation          | Пулы для DB и Kafka изолированы и всегда имеют гарантированный доступ к ресурсам   |
| «Зависание» сервиса | Внешний провайдер не может «съесть» все потоки и заблокировать внутренние операции |
| Потеря платежей     | Асинхронные потоки продолжают работу даже при деградации внешних систем            |


---

# Учитываемые отказы и способы обработки

Падение одного инстанса сервиса:
- Kubernetes автоматически перезапускает pod.
- Трафик перераспределяется балансировщиком.

Временная недоступность платежного провайдера:
- Защита:
    - таймауты;
    - retries с backoff;
    - circuit breaker.
- Fallback:
    - платеж переводится в PENDING;
    - дальнейшая обработка асинхронно через очередь;
    - клиент не получает мгновенный FAILED.

Рост нагрузки на Read-модель:
- Redis снижает нагрузку на ClickHouse.
- Возможно ограничить rate limit для тяжёлых запросов.