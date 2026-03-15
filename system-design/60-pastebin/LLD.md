# LLD: Pastebin

## APIs

```http
POST   /v1/pastes   Body: { content, expiry: "10m"|"1h"|"1d"|"never", custom_slug?, password? }
  Response: { slug, url, expires_at }

GET    /:slug   → raw content or HTML page (syntax highlight optional)
GET    /:slug/raw   → raw only
```

## Key Classes

```text
PasteService – create(content, expiry, customSlug?) → slug; get(slug) → content | 404
SlugGenerator – generate() → random 8-char; or base62(counter)
PasteRepository – save(slug, content, expiresAt); get(slug) → paste | null; deleteExpired() cron
```

## Expiry

- expiry "10m" → expires_at = now + 10 min. On get: if now > expires_at, return 404 and optionally delete. Cron: DELETE FROM pastes WHERE expires_at < NOW().

## Error Handling

- Duplicate custom_slug: 409. Content too large: 413. Invalid expiry: 400.

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

