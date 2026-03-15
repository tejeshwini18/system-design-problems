# Low-Level Design: Payment Gateway

## 1. APIs

```http
POST   /v1/payments
  Headers: Idempotency-Key: <key>
  Body: { "amount": 1000, "currency": "INR", "payment_method": "card_token_xxx", "capture": true, "metadata": {} }
  Response: { "id": "pay_123", "status": "captured", "amount", "created_at" }

GET    /v1/payments/:id
  Response: { "id", "status", "amount", "refunded_amount", ... }

POST   /v1/payments/:id/capture
  Headers: Idempotency-Key: <key>
  Body: { "amount": 1000 }   // optional partial capture
  Response: { "id", "status": "captured" }

POST   /v1/refunds
  Headers: Idempotency-Key: <key>
  Body: { "payment_id": "pay_123", "amount": 500 }
  Response: { "id": "ref_456", "status": "succeeded" }

GET    /v1/refunds/:id
```

---

## 2. Database Schemas

```sql
transactions (
  id PK, merchant_id FK, idempotency_key UNIQUE,
  amount_cents BIGINT, currency VARCHAR(3), status ENUM(pending, authorized, captured, failed, refunded),
  payment_method_type VARCHAR(20), external_id VARCHAR(100),  -- acquirer reference
  metadata JSON, created_at, updated_at
);
CREATE UNIQUE INDEX idx_idempotency ON transactions(merchant_id, idempotency_key) WHERE idempotency_key IS NOT NULL;

refunds (
  id PK, transaction_id FK, amount_cents BIGINT, status ENUM(pending, succeeded, failed),
  idempotency_key UNIQUE, external_id, created_at
);

webhook_deliveries (
  id PK, transaction_id, event_type, payload JSON, url TEXT, status ENUM(pending, delivered, failed),
  attempts INT, next_retry_at, created_at
);
```

---

## 3. Key Classes / Modules

```text
PaymentService
  - createPayment(merchantId, idempotencyKey, amount, currency, paymentMethod, capture) → payment
  - getPayment(paymentId) → payment
  - capturePayment(paymentId, idempotencyKey, amount?) → payment
  - createRefund(paymentId, idempotencyKey, amount) → refund

IdempotencyService
  - get(key) → cached response | null
  - set(key, response, ttl)

AcquirerClient (interface)
  - authorize(paymentId, amount, currency, token) → result
  - capture(paymentId, amount) → result
  - refund(refundId, paymentId, amount) → result

WebhookDispatcher
  - dispatch(paymentId, eventType, payload)
  - retry(): poll webhook_deliveries where status=pending and next_retry_at <= now
```

---

## 4. Create Payment (Steps)

1. Extract Idempotency-Key; if missing return 400.
2. IdempotencyService.get(merchant_id + idempotency_key). If hit, return cached response (same status code and body).
3. Begin DB transaction: insert transaction (status=pending), commit.
4. Call AcquirerClient.authorize(...). On success, if capture=true call AcquirerClient.capture(...).
5. Update transaction status (captured/failed); IdempotencyService.set(key, response, 24h).
6. WebhookDispatcher.dispatch(payment_id, "payment.succeeded" or "payment.failed", payload).
7. Return response to client.

---

## 5. Webhook Retry Policy

- Exponential backoff: 1m, 5m, 30m, 2h, 24h (e.g. 5 attempts).
- Store payload and HMAC signature; merchant verifies using shared secret.
- On 2xx from merchant, mark delivered; else increment attempts and set next_retry_at.

---

## 6. Idempotency Key Scope

- Per operation type: e.g. create payment key ≠ capture key.
- Key = (merchant_id, idempotency_key) to avoid cross-merchant reuse.
- Store full response (status + body) so retry returns exact same response.

---

## 7. Error Handling

- Duplicate idempotency key with different body: 422 or return existing (same key = same logical request).
- Acquirer timeout: keep transaction pending; async job to poll acquirer status or retry; eventually mark failed and webhook.
- Refund amount > captured: 400.

---

## Interview-Readiness Enhancements

### API and consistency
- Mark idempotency requirements for mutation APIs.
- Specify pagination/cursor strategy for list endpoints.
- Clarify consistency guarantees per endpoint/workflow.

### Data model and concurrency
- Explicitly list partition key/index choices and why.
- State optimistic vs pessimistic locking policy and conflict handling.
- Define deduplication/idempotent-consumer strategy for async paths.

### Reliability and operations
- Add explicit failure scenarios with mitigations and degradation behavior.
- Add monitoring/alert thresholds for critical flows and queue lag.
- Document rollout and rollback steps for schema/API changes.

### Validation checklist
- Include unit + integration + load + failure-injection test cases for critical paths.

