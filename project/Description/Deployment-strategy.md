# Deployment Strategy

## Общая модель

Используется GitOps-подход:
- GitLab CI отвечает за сборку и публикацию образа
- Kubernetes-манифесты хранятся в отдельном Git-репозитории
- Argo CD синхронизирует состояние кластера с Git
- Argo Rollouts управляет стратегией деплоя

CI **не имеет kubectl-доступа к кластеру**.

---

## Rollout Strategy

### Canary deployment

Используем canary-развёртывание с поэтапным увеличением трафика:

| Шаг | Трафик | Пауза                   |
|-----|--------|-------------------------|
| 1   | 10%    | 2 минуты                |
| 2   | 50%    | 5 минут                 |
| 3   | 100%   | после успешного анализа |

---

### Automated analysis

Перед переводом на 100% выполняется анализ метрик:
- error rate
- latency p95
- HTTP 5xx

Источник метрик: Prometheus.

При нарушении SLO:
- rollout автоматически прерывается
- происходит откат на stable revision

---

## Traffic Routing

- Ingress / API Gateway поддерживает weighted routing
- Argo Rollouts управляет весами
- Пользовательский трафик прозрачно распределяется между версиями

---

## Health checks

### Readiness
- `/actuator/health/readiness`
- сервис не получает трафик до полной готовности

### Liveness
- `/actuator/health/liveness`
- автоматический рестарт при зависании

---

## Graceful shutdown

- `preStop: sleep 20s`
- `terminationGracePeriodSeconds: 30`
- позволяет:
    - завершить HTTP-запросы
    - корректно закрыть Kafka consumers
    - избежать потери сообщений

---

## Rollback

Rollback возможен:
- автоматически (по результатам analysis)
- вручную