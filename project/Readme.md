- [Текстовая секция "Контекст, границы и коммуникации"](Description/Description_1.md)
- [Context диаграмма](C4/Context.puml)
- [C4 диаграмма](C4/BankPlatform_C2.puml)

---

- [Текстовая секция "Данные и консистентность"](Description/Description_2.md)
- [C4 диаграмма](C4/BankPlatform_Outbox_C2.puml)
- [C4 диаграмма Transaction Service](C4/TransactionService.puml)
- [C4 диаграмма Wallet Service](C4/WalletService.puml)
- [C4 диаграмма Query Service](C4/QueryService.puml)
- [ERD диаграмма](Erd/Erd.puml)
- [Sequence диаграмма - сценарий "успешный платеж"](Sequence/SequenceSuccess.puml)
- [Sequence диаграмма - сценарий "неуспешный платеж"](Sequence/SequenceFail.puml)

---

- [Текстовая секция "Надежность, масштабирование и наблюдаемость"](Description/Description_3.md)
- [C4 диаграмма Cache](C4/BankPlatform_Cache_C2.puml)
- [Sequence-диаграмма сценария: "клиент открывает историю платежей"](Sequence/SequenceCache.puml)
- [C4 диаграмма Основные топики](C4/Dql_topics_C2.puml)
- [Sequence-диаграмма сценария: "сообщение не удалось обработать несколько раз -> попадание в DLQ -> дальнейшая ручная/отложенная обработка"](Sequence/Sequence_Topic_to_dql_C2.puml)
- [Sequence диаграмма - сценарий "провайдер тормозит/падает"](Sequence/SequenceProvider.puml)
- [C4 диаграмма Observability](C4/BankPlatform_Observability_C2.puml)

---
- [C4 Container - AuthN/AuthZ](C4/BankPlatform_Security_C2.puml)
- [Sequence - OIDC Authorization Code Flow](Sequence/SequenceOIDSFlow.puml)
- [Service-to-service security + 12-facto](Description/Security-auth.md)
- [C4 Container - Webhook security](C4/BankPlatform_WebHook_Security_C2.puml)
- [Sequence - Webhook validation + anti-replay + idempotency](Sequence/WebhookValidation.puml)
- [Webhook/Callback security: подпись + timestamp + anti-replay + idempotency](Description/Webhooks-security.md)
- [C4 Container - Vault + injection](C4/BankPlatform_Vault_C2.puml)
- [Sequence - Vault agent + rotation](Sequence/SequenceVaultAgent.puml)
- [Secrets: Vault / K8s Secrets и ротация](Description/Secrets-management.md)
- [C4 Container - CI/CD + Argo Rollouts + Gateway](C4/C4_cicd_argo_rollouts.puml)
- [Sequence - canary/blue-green rollout + rollback](Sequence/SequenceCanaryRollout.puml)
- [Safe deployments: Blue-Green/Canary через Argo Rollouts + GitLab CI](Description/Db-migration-feature-toggle.md)
- [Deployment Strategy](Description/Deployment-strategy.md)
- [Release Checklist](Description/Release-checklist.md)
- [Runbook](Description/Runbook.md)
- [C4 Container - feature toggle migration](C4/Db_migration_feature_toggle.puml)
- [Sequence - expand/contract migration](Sequence/Sequence_db_migration_expand.puml)
- [Sequence - M2M](SequenceMerchant.puml)
---

- [.gitlab-ci.yml](config/.gitlab-ci.yml)
- [rollout.yaml](config/rollout.yaml)
- [payment-error-rate.yaml](config/payment-error-rate.yaml)