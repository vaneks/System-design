# Webhook/Callback security: подпись + timestamp + anti-replay + idempotency

## Проверка подписи webhook (на Gateway)

Заголовки:
- X-Signature: HMAC-SHA256
- X-Timestamp: unix_epoch_seconds
- X-Event-Id: uuid

HMAC(secret, timestamp + "." + raw_body)

## Anti-Replay защита

| Механизм	    | Описание   |
|--------------|------------|
| Time window  | 	±5 минут  |
| Replay Store | 	Redis     |
| Ключ         | 	eventId   |
| TTL          | 	10 минут  |

## Идемпотентность в Transaction Service

Callback содержит providerPaymentId

Перед обновлением статуса:
- проверяем текущий статус
- финальный статус не перезаписывается

Повторный callback:
- ACK без side effects
- PaymentResult не публикуется повторно