# Описание API контрактов

## Создание платежа

POST rest/v1/payment

**Request**

header : Authorization: Bear eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.KMUFsIDTnFmyG3nMiGM6H9FNFUROf3wh7SmqJp-QV30eyJraWQiOiIxZWQzOGYyZC03ZDM5LTZkMTUtYjM5Yy01M2M2OTI5YzQ1NmYiLCJ0eXAiOiJiZWFyZXIiLCJhbGciOiJSUzI1NiJ9.eyJhdWQiOiJmZmlucGF5LnJ1Iiwic3ViIjoiNTE5ODc3IiwibmJmIjoxNzY0ODY3NTUyLCJwaG9uZSI6Ijc3Nzc3NzAwMDMiLCJpc3MiOiJNTyIsInR5cGUiOiJBQ0NFU1MiLCJleHAiOjE3NjQ4NjkzNTIsInZlcnNpb24iOiI4LjExLjMiLCJpYXQiOjE3NjQ4Njc1NTIsImp0aSI6IjFmMGQxMzI4LWYxOTgtNjJhOS04YmIyLTkzNTM1YjJiZjFhYyIsInVjaWQiOiJhMjA4ZTc0MC1jYTc5LTExZWUtODQ4OC02OTVjMDNlMzdmODgifQ.dChrimf9-C4D2PyJM35cMrwaoCpZjLiF78snAUbleVNIsO_wczEPc2i0pUw0s-f0CCeQsl629Rf_iB3VCQlmk6H2E8RSGkMpZVvlUQcEgjtNoUkHCmMLOHtxu1icBKe-z3UtzMSB1_buxFBoviMyVoD30suUq559DQQJ14rdnrrlyxsiHg7-8RqWeQyqmtayT4esLSXEuCuNSOXE7cWTnRhCAAvCnn2jDZMFpg9eq_QxgEh1vVygy8PNmD9Vmck8sAfDjRh1CnxOBSUXSyzyaT1jpVRv9GX26WCBiRQujKnd4QbslDxO8W0u9Xe_561IMTNwMLBLvdhmNb-XMl7fgA

{
    "snd_account_number": "232323232323233223",
    "rcv_account_number": "232323232323233222",
    "amount": 150.00,
    "currency": "USD"
}

**Response**

{
    "paymentId": "019b03ab-08e9-772d-9f15-b43d6f185341",
    "status": "PENDING"
}


## Создание платежа в External Payment Provider 

POST https://gc.com/rest/v1/confirm

**Request**

{
    "paymentId": "019b03ab-08e9-772d-9f15-b43d6f185341",
    "snd_account_number": "232323232323233223",
    "rcv_account_number": "232323232323233222",
    "amount": 150.00,
    "currency": "USD"
}

**Response**

{
    "paymentId": "019b03ab-08e9-772d-9f15-b43d6f185341",
    "status" : "ACCEPTED"
}

## Получение статуса платежа от External Payment Provider

POST rest/v1/callback/result

**Request**

{
    "paymentId": "019b03ab-08e9-772d-9f15-b43d6f185346",
    "providerId" : "055b03ab-08e9-772d-9f15-b43d6f185000",
    "status": "SUCCESS",
    "timestamp": "2025-12-10:00:00Z"
}

**Response**

{
    "status": "RESERVED",
    "balance": 840.00
}

## Получение информации о платеже

**Request**

header : Authorization: Bear eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.KMUFsIDTnFmyG3nMiGM6H9FNFUROf3wh7SmqJp-QV30eyJraWQiOiIxZWQzOGYyZC03ZDM5LTZkMTUtYjM5Yy01M2M2OTI5YzQ1NmYiLCJ0eXAiOiJiZWFyZXIiLCJhbGciOiJSUzI1NiJ9.eyJhdWQiOiJmZmlucGF5LnJ1Iiwic3ViIjoiNTE5ODc3IiwibmJmIjoxNzY0ODY3NTUyLCJwaG9uZSI6Ijc3Nzc3NzAwMDMiLCJpc3MiOiJNTyIsInR5cGUiOiJBQ0NFU1MiLCJleHAiOjE3NjQ4NjkzNTIsInZlcnNpb24iOiI4LjExLjMiLCJpYXQiOjE3NjQ4Njc1NTIsImp0aSI6IjFmMGQxMzI4LWYxOTgtNjJhOS04YmIyLTkzNTM1YjJiZjFhYyIsInVjaWQiOiJhMjA4ZTc0MC1jYTc5LTExZWUtODQ4OC02OTVjMDNlMzdmODgifQ.dChrimf9-C4D2PyJM35cMrwaoCpZjLiF78snAUbleVNIsO_wczEPc2i0pUw0s-f0CCeQsl629Rf_iB3VCQlmk6H2E8RSGkMpZVvlUQcEgjtNoUkHCmMLOHtxu1icBKe-z3UtzMSB1_buxFBoviMyVoD30suUq559DQQJ14rdnrrlyxsiHg7-8RqWeQyqmtayT4esLSXEuCuNSOXE7cWTnRhCAAvCnn2jDZMFpg9eq_QxgEh1vVygy8PNmD9Vmck8sAfDjRh1CnxOBSUXSyzyaT1jpVRv9GX26WCBiRQujKnd4QbslDxO8W0u9Xe_561IMTNwMLBLvdhmNb-XMl7fgA

GET rest/v1/payment/{id}

**Response**

{
    "paymentId" : "019b03ab-08e9-772d-9f15-b43d6f185341",
    "status": "SUCCESS",
    "amount": 840.00,
    "currency": "USD",
    "snd_account_number": "232323232323233223",
    "rcv_account_number": "232323232323233222",
    "timestamp": "2025-12-10:00:00Z"
}

# Формат сообщений для kafka

## PaymentInitiated

{
    "paymentId": "019b03ab-08e9-772d-9f15-b43d6f185341",
    "personId": "019b03ab-08e9-772d-9f15-b43d6f185342",
    "walletId": "019b03ab-08e9-772d-9f15-b43d6f185340",
    "amount": 150.00,
    "currency": "USD",
    "snd_account_number": "232323232323233223",
    "rcv_account_number": "232323232323233222",
    "timestamp": "2025-12-10:00:00Z"
}

## PaymentResult

{
    "paymentId": "019b03ab-08e9-772d-9f15-b43d6f185346",
    "status": "SUCCESS",
    "timestamp": "2025-12-10:00:00Z"
}

{
    "paymentId": "019b03ab-08e9-772d-9f15-b43d6f185346",
    "status": "FAILED",
    "errorCode": "202022",
    "errorMessage" : "Истекло время подтверждения платежа"
    "timestamp": "2025-12-10:00:00Z"
}

## Описание процесса исполнения платежа

- Из МП вызывается рест: **POST rest/v1/payment** c header, в котором содержится JWT-токен.
- Пройдя валидацию и проверку токена в **API Gateway**, запрос попадает в Wallet Service.
- В Wallet Service делаем проверку на дедупликацию.
- Из токена извлекается person_id, производится проверка пользователя (существует/в статусе ACTIVE). Аналогично проверяем snd_account_number.
- Проверяем баланс, резервируем денежные средства и создаем запись в wallet_transactions в статусе Pending.
- Формируем сообщение и с помощью паттерна Transactional Outbox производим отправку в МС Transactional Service/Payment Query Service.
- В Payment Query Service обрабатываем сообщение и пишем в БД.
- В Transactional Service при получении сообщения из kafka, создаем запись в бд (таблица payments) в статусе Pending.
- Формируем сообщение и производим отправку в Callback Service, после чего меняем статус на Sent.
- В Callback Service производим запись сообщения в БД (callback_payments).
- Формируем сообщение и производим отправку в External Payment Provider по средствам вызова реста: POST https://gc.com/rest/v1/confirm. В случае недоступности, у нас должен быть настроен механизм ретраев. Получив статус ответа (ACCEPTED/FAILED), можно сделать отправку нотификации клиенту (websocket/push).
- Через некоторое время, External Payment Provider инициирует вызов реста: POST rest/v1/callback/result. Получив информацию от External Payment Provider, обновляем данные в БД и с помощью паттерна Transactional Outbox производим отправку в МС Transactional Service.
- В Transactional Service обновляем статус платежа. И далее формируем и отправляем сообщение с помощью паттерна Transactional Outbox в Wallet Service/Payment Query Service..
- В Payment Query Service обновляем статус платежа.
- В Wallet Service, в случае, если статус SUCCESS: обновляем статус платежа на Completed, подтверждаем списание.
- Если статус FAILED: обновляем статус платежа на Failed, производим rollback.
- Далее отправляем клиенту уведомление об успешном/неуспешном платеже (опционально).

- Из МП вызывается рест **GET rest/v1/payment/{id}** c header, в котором содержится JWT-токен.
- Пройдя валидацию и проверку токена в **API Gateway**, запрос попадает в Payment Query Service.
- Из токена извлекается person_id, производится проверка пользователя (существует/в статусе ACTIVE). Далее проверяем принадлежность платежа клиенту.
- Если платеж не принадлежит клиенту - отдаем 403. Иначе формируем ответ.

## Механизмы дедупликаций

## Wallet Service

- **При вызове реста: POST rest/v1/payment**
Из полей сообщения snd_account_number + rcv_account_number + amount + currency + person_id(извлекаем из токена) формируем hash.
Проверяем наличие hash в Redis (ttl_hash - выносим в property), если запись есть - отдаем ошибку (реализуем с помощью Resilience4j).
Если нет, то пишем в Redis hash, и далее по сценарию.

- **При получении сообщения из kafka:**
Проверяем наличие записи в таблице wallet_transactions по полям: paymentId + status.
Если есть, то игнорируем.

## Transactional Service

- **При вызове реста: POST rest/v1/callback/result**
Проверяем наличие записи в таблице callback_payments по provider_id.
Если есть, то игнорируем + отправка e-mail (для разбора инцидента).

- **При получении сообщения из kafka:**
Проверяем наличие записи в таблице callback_payments по wallet_id.
Если есть, то игнорируем.