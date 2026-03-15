# Low-Level Design: Notification System

## 1. APIs

```http
POST   /v1/notifications/send
  Body: {
    "user_id": "u123",
    "channel": "email" | "sms" | "push",
    "template_id": "order_shipped",
    "variables": { "order_id": "o456", "name": "John" },
    "idempotency_key": "order_o456_shipped"
  }
  Response: { "notification_id": "n789", "status": "queued" }

GET    /v1/notifications/:id
  Response: { "id", "status", "channel", "created_at", "delivered_at" }

GET    /v1/users/:id/notifications
  Query: limit, cursor (for in-app inbox)
  Response: { "items": [...], "next_cursor": "..." }

PUT    /v1/users/:id/preferences
  Body: { "email": true, "sms": false, "push": true, "categories": { "marketing": false } }
```

---

## 2. Database Schemas

```sql
user_preferences (user_id PK, channel, category, enabled BOOLEAN, updated_at)
  UNIQUE(user_id, channel, category)

templates (template_id PK, channel, name, subject, body, variables_schema JSON, created_at)

notifications (
  id PK, user_id, channel, template_id, payload JSON, status ENUM(queued, sent, delivered, failed),
  idempotency_key UNIQUE, created_at, delivered_at, provider_response TEXT
)
CREATE INDEX idx_user_created ON notifications(user_id, created_at DESC);
CREATE INDEX idx_idempotency ON notifications(idempotency_key);
```

---

## 3. Key Classes / Modules

```text
NotificationService
  - send(userId, channel, templateId, variables, idempotencyKey?) → notificationId
  - getStatus(notificationId) → status
  - listByUser(userId, limit, cursor) → items, nextCursor

PreferenceService
  - get(userId) → preferences
  - canSend(userId, channel, category) → boolean
  - update(userId, preferences)

TemplateService
  - get(templateId) → template
  - render(templateId, variables) → { subject?, body }

ChannelWorkers
  - EmailWorker: consume email queue → SES
  - SmsWorker: consume sms queue → Twilio
  - PushWorker: consume push queue → FCM/APNs

IdempotencyChecker
  - check(key) → notificationId | null
  - set(key, notificationId, ttl)
```

---

## 4. Send Flow (Steps)

1. Resolve template and variables; validate required variables.
2. Check preferences: if channel or category disabled, return without sending (or return 200 with status "skipped").
3. Idempotency: get(idempotency_key); if present, return that notification_id.
4. Insert notification row with status=queued; set idempotency_key → id in Redis (TTL 24h).
5. Publish to channel queue: { notification_id, user_id, channel, rendered_subject, rendered_body, device_tokens (push) }.
6. Return notification_id and status=queued.
7. Worker: fetch from queue; call provider; on success update status=delivered and delivered_at; on failure requeue with delay or send to DLQ.

---

## 5. Template Rendering

- Template body: "Hi {{name}}, your order {{order_id}} has shipped."
- variables_schema: ["name", "order_id"] for validation.
- Render: simple replace {{key}} with variables[key]; escape for HTML in email.

---

## 6. Retry Policy (Worker)

- Retry up to 5 times: delay 1s, 2s, 4s, 8s, 16s (exponential backoff).
- After 5 failures: move to DLQ; set status=failed; alert.
- Idempotency: use notification_id so provider call is not duplicated (e.g. store provider_id in notifications).

---

## 7. Error Handling

- Invalid template_id: 400.
- User not found / no device token for push: skip or 400.
- Provider rate limit: 429 from provider → requeue with longer delay.
- Provider down: retry with backoff; circuit breaker per provider.

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

