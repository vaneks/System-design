# Описание API контрактов

## Создание платежа

POST rest/v1/payment

**Request**

header : Authorization: Bear eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.KMUFsIDTnFmyG3nMiGM6H9FNFUROf3wh7SmqJp-QV30eyJraWQiOiIxZWQzOGYyZC03ZDM5LTZkMTUtYjM5Yy01M2M2OTI5YzQ1NmYiLCJ0eXAiOiJiZWFyZXIiLCJhbGciOiJSUzI1NiJ9.eyJhdWQiOiJmZmlucGF5LnJ1Iiwic3ViIjoiNTE5ODc3IiwibmJmIjoxNzY0ODY3NTUyLCJwaG9uZSI6Ijc3Nzc3NzAwMDMiLCJpc3MiOiJNTyIsInR5cGUiOiJBQ0NFU1MiLCJleHAiOjE3NjQ4NjkzNTIsInZlcnNpb24iOiI4LjExLjMiLCJpYXQiOjE3NjQ4Njc1NTIsImp0aSI6IjFmMGQxMzI4LWYxOTgtNjJhOS04YmIyLTkzNTM1YjJiZjFhYyIsInVjaWQiOiJhMjA4ZTc0MC1jYTc5LTExZWUtODQ4OC02OTVjMDNlMzdmODgifQ.dChrimf9-C4D2PyJM35cMrwaoCpZjLiF78snAUbleVNIsO_wczEPc2i0pUw0s-f0CCeQsl629Rf_iB3VCQlmk6H2E8RSGkMpZVvlUQcEgjtNoUkHCmMLOHtxu1icBKe-z3UtzMSB1_buxFBoviMyVoD30suUq559DQQJ14rdnrrlyxsiHg7-8RqWeQyqmtayT4esLSXEuCuNSOXE7cWTnRhCAAvCnn2jDZMFpg9eq_QxgEh1vVygy8PNmD9Vmck8sAfDjRh1CnxOBSUXSyzyaT1jpVRv9GX26WCBiRQujKnd4QbslDxO8W0u9Xe_561IMTNwMLBLvdhmNb-XMl7fgA
header: Idempotency-Key: 6c2b7b4e-6d6a-4c7b-b3b1-9f7a9f3e6c91
header: X-Correlation-Id: 3f0b9a61-41c7-4a8b-9a6b-1db4eaa4f123

{
    "snd_account_number": "232323232323233223",
    "rcv_account_number": "232323232323233222",
    "amount": 150.00,
    "currency": "USD"
}

**Response**

{
    "paymentId": "019b03ab-08e9-772d-9f15-b43d6f185341",
    "status": "PENDING",
    "correlationId": "3f0b9a61-41c7-4a8b-9a6b-1db4eaa4f123",   
    "createTime": "2025-12-14T04:30:49.493Z"
}


## Создание платежа в External Payment Provider 

POST https://gc.com/rest/v1/confirm

header: X-Correlation-Id: 3f0b9a61-41c7-4a8b-9a6b-1db4eaa4f123

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
    "providerId" : "055b03ab-08e9-772d-9f15-b43d6f185000",
    "correlationId": "3f0b9a61-41c7-4a8b-9a6b-1db4eaa4f123",        
    "status" : "ACCEPTED"
    "createTime": "2025-12-14T04:30:49.493Z"
}

## Получение статуса платежа от External Payment Provider

POST rest/v1/callback/result

**Request**

header: X-Correlation-Id: 3f0b9a61-41c7-4a8b-9a6b-1db4eaa4f123

{
    "paymentId": "019b03ab-08e9-772d-9f15-b43d6f185346",
    "providerId" : "055b03ab-08e9-772d-9f15-b43d6f185000",
    "createTime": "2025-12-14T04:30:49.493Z"
}

**Response**

{
    "paymentId": "019b03ab-08e9-772d-9f15-b43d6f185346",
    "providerId" : "055b03ab-08e9-772d-9f15-b43d6f185000",
    "correlationId": "3f0b9a61-41c7-4a8b-9a6b-1db4eaa4f123",    
    "status": "ACCEPTED"
}

## Получение информации о платеже

**Request**

header: Authorization: Bear eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.KMUFsIDTnFmyG3nMiGM6H9FNFUROf3wh7SmqJp-QV30eyJraWQiOiIxZWQzOGYyZC03ZDM5LTZkMTUtYjM5Yy01M2M2OTI5YzQ1NmYiLCJ0eXAiOiJiZWFyZXIiLCJhbGciOiJSUzI1NiJ9.eyJhdWQiOiJmZmlucGF5LnJ1Iiwic3ViIjoiNTE5ODc3IiwibmJmIjoxNzY0ODY3NTUyLCJwaG9uZSI6Ijc3Nzc3NzAwMDMiLCJpc3MiOiJNTyIsInR5cGUiOiJBQ0NFU1MiLCJleHAiOjE3NjQ4NjkzNTIsInZlcnNpb24iOiI4LjExLjMiLCJpYXQiOjE3NjQ4Njc1NTIsImp0aSI6IjFmMGQxMzI4LWYxOTgtNjJhOS04YmIyLTkzNTM1YjJiZjFhYyIsInVjaWQiOiJhMjA4ZTc0MC1jYTc5LTExZWUtODQ4OC02OTVjMDNlMzdmODgifQ.dChrimf9-C4D2PyJM35cMrwaoCpZjLiF78snAUbleVNIsO_wczEPc2i0pUw0s-f0CCeQsl629Rf_iB3VCQlmk6H2E8RSGkMpZVvlUQcEgjtNoUkHCmMLOHtxu1icBKe-z3UtzMSB1_buxFBoviMyVoD30suUq559DQQJ14rdnrrlyxsiHg7-8RqWeQyqmtayT4esLSXEuCuNSOXE7cWTnRhCAAvCnn2jDZMFpg9eq_QxgEh1vVygy8PNmD9Vmck8sAfDjRh1CnxOBSUXSyzyaT1jpVRv9GX26WCBiRQujKnd4QbslDxO8W0u9Xe_561IMTNwMLBLvdhmNb-XMl7fgA
header: X-Correlation-Id: 3f0b9a61-41c7-4a8b-9a6b-1db4eaa4f123

GET rest/v1/payment/{id}

**Response**

{
    "paymentId" : "019b03ab-08e9-772d-9f15-b43d6f185341",
    "correlationId": "3f0b9a61-41c7-4a8b-9a6b-1db4eaa4f123",   
    "status": "SUCCESS",
    "amount": 840.00,
    "currency": "USD",
    "snd_account_number": "232323232323233223",
    "rcv_account_number": "232323232323233222",
    "createTime": "2025-12-14T04:30:49.493Z",
    "updateTime": "2025-12-14T04:30:49.493Z"
}

# Формат сообщений для kafka

## PaymentInitiated

{
    "paymentId": "019b03ab-08e9-772d-9f15-b43d6f185341",
    "eventId": "019b03ab-08e9-772d-9f15-b43d6f185341",
    "schemaVersion": 1,
    "personId": "019b03ab-08e9-772d-9f15-b43d6f185342",
    "walletId": "019b03ab-08e9-772d-9f15-b43d6f185340",
    "correlationId": "3f0b9a61-41c7-4a8b-9a6b-1db4eaa4f123",
    "amount": 150.00,
    "currency": "USD",
    "snd_account_number": "232323232323233223",
    "rcv_account_number": "232323232323233222",
    "createTime": "2025-12-14T04:30:49.493Z"
}

## PaymentResult

{
    "paymentId": "019b03ab-08e9-772d-9f15-b43d6f185346",
    "eventId": "019b03ab-08e9-772d-9f15-b43d6f185341",
    "schemaVersion": 1,
    "correlationId": "3f0b9a61-41c7-4a8b-9a6b-1db4eaa4f123",   
    "status": "SUCCESS",
    "createTime": "2025-12-14T04:30:49.493Z"
}

{
    "paymentId": "019b03ab-08e9-772d-9f15-b43d6f185346",
    "eventId": "019b03ab-08e9-772d-9f15-b43d6f185341",
    "schemaVersion": 1,
    "status": "FAILED",
    "errorCode": "202022",
    "errorMessage" : "Истекло время подтверждения платежа",
    "createTime": "2025-12-14T04:30:49.493Z"
}

## Статусная модель

| Статус         | Описание                                          |
|----------------|---------------------------------------------------|
| NEW            | Платеж создан, но средства еще не зарезервированы |
| PENDING        | Средства зарезервированы                          |
| SENT           | Запрос отправлен платежному провайдеру            |
| ACCEPTED       | Платежный провайдер принял платеж в обработку     |
| SUCCESS        | Деньги списаны окончательно                       |
| FAILED         | Платеж отклонен                                   |
| FAILED_TIMEOUT | Платеж отклонен по таймауту                       |
 


## Описание процесса исполнения платежа

- Из МП вызывается рест: **POST rest/v1/payment** c header, в котором содержится JWT-токен.
- Пройдя валидацию и проверку токена в **API Gateway**, запрос попадает в Wallet Service.
- В Wallet Service создаем платеж в статусе NEW.
- Из токена извлекается person_id, производится проверка пользователя (существует/в статусе ACTIVE). Аналогично проверяем snd_account_number.
- Проверяем баланс, резервируем денежные средства и создаем запись в wallet_transactions в статусе PENDING.
- Формируем сообщение и с помощью паттерна Transactional Outbox производим отправку в МС Transactional Service/Payment Query Service.
- В Payment Query Service обрабатываем сообщение и пишем в БД.
- В Transactional Service при получении сообщения из kafka, создаем запись в бд (таблица payments) в статусе PENDING.
- Формируем сообщение и производим отправку в Payment Service, после чего меняем статус на SENT.
- В Payment Service производим запись сообщения в БД.
- Формируем сообщение и производим отправку в External Payment Provider по средствам вызова реста: POST https://gc.com/rest/v1/confirm. В случае недоступности, у нас должен быть настроен механизм ретраев. Получив статус ответа (ACCEPTED/FAILED), можно сделать отправку нотификации клиенту (websocket/push).
- Через некоторое время, External Payment Provider инициирует вызов реста: POST rest/v1/callback/result. Получив информацию от External Payment Provider, обновляем данные в БД и с помощью паттерна Transactional Outbox производим отправку в МС Transactional Service.
- В Transactional Service обновляем статус платежа. И далее формируем и отправляем сообщение с помощью паттерна Transactional Outbox в Wallet Service/Payment Query Service. Если ответ не пришел в течении 3 сек, меняем статус на FAILED_TIMEOUT и и отправляем сообщение с помощью паттерна Transactional Outbox в Wallet Service/Payment Query Service.
- В Payment Query Service обновляем статус платежа.
- В Wallet Service, в случае, если статус SUCCESS: обновляем статус платежа на Completed, подтверждаем списание.
- Если статус FAILED/FAILED_TIMEOUT: обновляем статус платежа на FAILED/FAILED_TIMEOUT, производим rollback.
- Далее отправляем клиенту уведомление об успешном/неуспешном платеже (опционально).

## Описание процесса получения информации по платежу
- Из МП вызывается рест **GET rest/v1/payment/{id}** c header, в котором содержится JWT-токен.
- Пройдя валидацию и проверку токена в **API Gateway**, запрос попадает в Payment Query Service.
- Из токена извлекается person_id, производится проверка пользователя (существует/в статусе ACTIVE). Далее проверяем принадлежность платежа клиенту.
- Если платеж не принадлежит клиенту - отдаем 403. Иначе формируем ответ.

## Идемпотентность в МС

## Wallet Service

- **При вызове реста: POST rest/v1/payment**
Используется заголовок: Idempotency-Key: <uuid>.
МП передаёт Idempotency-Key. Ищем запись по idempotency_key.
- **Если запись найдена:**
возвращает сохранённый response (тот же paymentId, статус и тело ответа);
новый платёж не создаётся.
- **Если запись не найдена:**
создаётся платёж;
формируется paymentId;
ответ сохраняется в wallet_idempotency;
ответ возвращается клиенту.


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