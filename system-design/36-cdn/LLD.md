# Low-Level Design: CDN

## 1. APIs (Control Plane)

```http
POST   /v1/purge                    Purge cache
  Body: { "urls": ["https://cdn.example.com/a.jpg"], "paths": ["/images/*"], "tags": ["product-123"] }
GET    /v1/cache/status?url=...     Check cache status (hit/miss, TTL)
```

- **Delivery:** No explicit API; user requests URL to CDN hostname; edge serves or fetches from origin.

---

## 2. Key Classes / Modules (Edge)

```text
EdgeHandler
  - onRequest(request): key = cacheKey(request); entry = cache.get(key); if entry: return entry.response; else: response = fetchOrigin(request); cache.set(key, response, ttl); return response

CacheKey
  - build(request) → string
  - default: request.host + request.path + optional(request.query)
  - optional: include headers (e.g. Accept-Language) for varied content

CacheStore (per edge)
  - get(key) → (response, expiry) | null
  - set(key, response, ttl)
  - delete(key); deleteByPrefix(prefix); deleteByTag(tag)
  - eviction: LRU when full; or TTL expiry

OriginFetcher
  - fetch(url, headers) → response
  - conditional: If-None-Match (ETag); 304 → use cached body
  - range: Range header → 206
```

---

## 3. Cache Key (Examples)

```text
key = host + path
  e.g. cdn.example.com/images/photo.jpg

key = host + path + sorted(query)
  e.g. cdn.example.com/api?id=1&user=2 → cdn.example.com/api?id=1&user=2
  (exclude utm_* for stability if desired)

key with tag: store key → tags[] for purge by tag
  e.g. key "cdn.example.com/p/123" → tags ["product-123", "category-5"]
```

---

## 4. TTL and Revalidation

- **Store:** (key, response_body, headers, expiry_time, etag).
- **On request:** If now < expiry_time, return cached (hit). If stale but etag present, send If-None-Match to origin; 304 → return cached body (revalidated); 200 → update cache and return.
- **Cache-Control parsing:** max-age=N → expiry = now + N; s-maxage for shared cache.

---

## 5. Purge (Steps)

1. **URL purge:** For each edge, delete(key) where key = url.
2. **Prefix purge:** For each key where key starts with prefix, delete(key); or maintain index prefix → keys and delete all.
3. **Tag purge:** For each key associated with tag, delete(key); edges receive purge request via control plane and evict.
4. **Propagation:** Purge request fan-out to all edges; complete in seconds to minutes.

---

## 6. Error Handling

- **Origin timeout:** Return 504 or serve stale (if configured).
- **Origin 5xx:** Retry (optional); serve stale or 502.
- **Cache full:** Evict LRU or lowest TTL; then set new entry.
- **Invalidation race:** After purge, next request may still get stale if in-flight; eventual consistency.

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

