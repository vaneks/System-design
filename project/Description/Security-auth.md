# OIDC Authorization Code Flow для UI-клиента

## Service-to-Service security + OIDC

# Секции:

- AuthN/AuthZ overview
- OIDC Authorization Code Flow
- Service-to-Service trust model
- Token types
- Scopes / roles
- Failure modes

| Caller      | Callee        | Token type  | Context | Где проверяется       | Scopes / Roles    |
|-------------|---------------|-------------|---------|-----------------------|-------------------|
| Mobile      | Wallet        | User JWT    | user    | Wallet Service        | payments.create   |
| Mobile      | Payment Query | User JWT    | user    | Payment Query Service | query.read        |
| Transaction | Wallet        | Service JWT | service | Wallet Service        | wallet.internal   |
| Transaction | Payment Query | Service JWT | service | Payment Query Service | query.internal    |
| Callback    | Transaction   | Service JWT | service | Transaction Service   | payments.callback |


# 12-Factor Configuration

Только через environment / secrets

Храним вне кода (secrets и configMaps Kubernetes):
- DB credentials
- OAuth client secrets
- Provider API keys
- Webhook signing secrets
- Feature flags
- Timeout / retry / CB параметры