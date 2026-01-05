# Runbook

## Webhook signature failures
**Symptoms:** 401 from gateway  
**Actions:**
1. Проверить Vault secret
2. Проверить timestamp skew
3. Проверить replay-store TTL

---

## Payment Provider down
**Symptoms:** рост PENDING_PROVIDER  
**Actions:**
1. Проверить CB state
2. Убедиться, что fallback включен
3. Оценить DLQ

---

## Consumer lag растет
**Symptoms:** lag > SLO  
**Actions:**
1. Проверить partitions
2. Увеличить consumer replicas
3. Проверить backpressure
