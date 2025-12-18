# Resilience-паттерны: Timeouts, Retries, Circuit Breaker, Bulkhead, Rate Limit, Fallback


| Взаимодействие                             | Таймаут | Retries                    | CB  | Bulkhead                | Rate Limit | Fallback  | Обоснование                                                                                                                                                          |
|--------------------------------------------|---------|----------------------------|-----|-------------------------|------------|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| BFF → Wallet Service                       | 1s      | нет                        | нет | Thread pool             | да         | нет       | Синхронный запрос от клиента, ожидается быстрый ответ. Rate limit защищает от клиентских всплесков и DDoS                                                            |
| Wallet → Transaction Service               | —       | Kafka retry + DLQ          | —   | Consumer group          | —          | DLQ       | Асинхронное взаимодействие через Kafka, at-least-once доставка, ошибки обрабатываются через DLQ                                                                      |
| Transaction → Payment Service              | —       | Kafka retry + DLQ          | —   | Consumer group          | —          | DLQ       | Асинхронная передача команд, устойчивость за счёт брокера сообщений                                                                                                  |
| Payment Service → Payment Provider         | 2s      | 3 (exp. backoff + jitter)  | да  | Отдельный   client pool | да         | PENDING   | Только в случае, если Payment Provider имеет защиту от двойных списаний!!! Внешний нестабильный сервис, нужен CB, retries с backoff, fallback в отложенную обработку |
| BFF → Payment Query Service                | 1s      | нет                        | нет | Thread pool             | да         | нет       | Read-only операции, предпочтительнее быстрый отказ                                                                                                                   |
| Notification → SMS Provider                | 3s      | 5 (exp. backoff + jitter)) | да  | Отдельный client pool   | да         | DLQ       | Внешний провайдер, допустима задержка доставки                                                                                                                       |
| API Gateway → Wallet/Payment Query Service | 1s      | нет                        | да  | Connection pool         | да         | 429 / 503 | Первая линия защиты: лимиты, таймауты, защита от каскадных отказов                                                                                                   |

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

| Сценарий                                                   | Основной топик     | DLQ-топик                  | Причина попадания в DLQ                        |
|------------------------------------------------------------|--------------------|----------------------------|------------------------------------------------|
| Transaction Service → Wallet Service/Payment Query Service | payments.result    | payments.transactional.dlq | Ошибка бизнес-валидации/несоответствие статуса |
| Wallet Service → Transaction Service/Payment Query Service | payments.initiated | payments.wallet.dlq        | Ошибка маппинга/неконсистентные данные         |
| Callback Service → Transaction Service                     | payments.callback  | provider.callback.dlq      | Невалидный payload                             |

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

---

# Observability: метрики, логи, трейсы, SLI/SLO

## Стек

Используем Prometheus + Grafana (метрики), Loki (логи), OpenTelemetry + Grafana Tempo(трейсы) для сбора метрик, логов и трейсов.

Prometheus:
- Стандарт для Kubernetes и микросервисов

Grafana:
- Универсальный UI для метрик, логов и трейсов.
- Поддержка SLO-дэшбордов.
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