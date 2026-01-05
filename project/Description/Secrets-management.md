# Secrets: Vault / K8s Secrets и ротация

# Типы секретов

| Секрет              | 	Где используется                    |
|---------------------|--------------------------------------|
| Provider API Key	   | Payment Service                      |
| Webhook Secret      | 	Gateway                             |
| OAuth Client Secret | 	Gateway / Services                  |
| DB Credentials	     | Wallet / Transaction / Payment Query |

# Хранение

Vault
- Vault + Kubernetes Auth
- Vault Agent Sidecar
- Secrets → files / env

Kubernetes Secrets
- Base64 encoded
- Mounted as env / volume

# Ротация

Webhook secret
- Gateway принимает:
  - current
  - previous
- Окно совместимости: 24h

Provider API Key
- Новый ключ добавляется в Vault
- Service reloads config
- Старый ключ отзывается


