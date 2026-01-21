## Текущее состояние (до миграции)

payments

| Поле                | 	Тип данных |
|---------------------|-------------|
| payment_id          | UUID        |
| person_id           | UUID        |
| wallet_id           | UUID        |
| amount	             | DECIMAL     |
| currency	           | VARCHAR     |
| snd_account_number  | VARCHAR     |
| rcv_account_number	 | VARCHAR     |
| status	             | VARCHAR     |
| created_at	         | TIMESTAMP   |
| updated_at	         | TIMESTAMP   |

## Целевое состояние (после миграции)

payments

| Поле                | 	Тип данных |
|---------------------|-------------|
| payment_id          | UUID        |
| person_id           | UUID        |
| wallet_id           | UUID        |
| amount	             | DECIMAL     |
| currency	           | VARCHAR     |
| snd_account_number  | VARCHAR     |
| rcv_account_number	 | VARCHAR     |
| status	             | VARCHAR     |
| status_reason	      | VARCHAR     |
| created_at	         | TIMESTAMP   |
| updated_at	         | TIMESTAMP   |

## Принципы миграции

- Zero-downtime — без остановки сервисов и блокирующих операций
- Backward compatible — старые версии сервисов продолжают работать
- Rollback через конфигурацию, без отката схемы БД
- Feature toggle управляет чтением, а не записью
- Schema first — сначала БД, потом код

# Этапы миграции (Expand / Contract)

## Expand — расширение схемы (без изменения поведения)

# Миграция БД

Liquibase:

ALTER TABLE payments
ADD COLUMN status_reason VARCHAR NULL;

# Обновление сервисов

Сервисы:
- Transaction Service
- Wallet Service
- Callback Service
- Payment Query Service

Записываем status_reason при наличии
Старое поведение сохраняется
Чтение status_reason ещё не используется


# Switch Read — переключение чтения (Feature Toggle)

Включаем флаг (payments.useStatusReason = true)

if (featureFlag.isEnabled("payments.useStatusReason")) { 
    return payment.getStatusReason();
} else {
    return mapLegacyStatus(payment.getStatus());
}

- Запись ведется в оба поля (status + status_reason)
- Флаг влияет только на read-path
- Флаг должен включаться динамически (без redeploy)

# Monitoring после включения

Обязательно контролируем:
- рост ошибок PaymentQuery
- расхождения статусов
- DLQ по событиям PaymentResult
- latency read-модели

# PromQL и пороги

Error rate
Порог:
- < 1% → ok
- > 2% → fail
- 1–2% → judgement

Latency (P95)
Порог:
- < 500ms → ok
- > 800ms → fail

# Удаление legacy-логики

Условия:
- флаг включён ≥ N дней
- нет rollback-ов
- аналитика и интеграции переведены

Действия:
- удалить legacy-маппинг
- оставить только status + status_reason
- столбец status не удаляем, он всё ещё основной

# Rollback Plan 

Сценарий: проблемы после включения флага
- Ошибки в API
- Некорректные статусы 
- Аналитика «падает»

Немедленный rollback (payments.useStatusReason = false)

# Finish

После тщательного тестирования:
- удаляем поле status
- оставляем return payment.getStatusReason();

# rollback boundary

DROP COLUMN

Код можно откатить, данные нет!
Поэтому:
- expand → migrate → contract
- write new / read old
- remove last





