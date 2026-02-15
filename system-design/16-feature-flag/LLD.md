# Low-Level Design: Feature Flag System

## 1. APIs

### Evaluation (high-throughput)
```http
POST   /v1/evaluate
  Body: { "context": { "user_id": "u123", "country": "US", "plan": "pro" }, "flags": ["new_checkout", "beta_search"] }
  Response: { "new_checkout": true, "beta_search": false }

GET    /v1/flags/:key/evaluate?user_id=u123&country=US&plan=pro
  Response: { "enabled": true }
```

### Admin
```http
GET    /v1/flags                    List flags
GET    /v1/flags/:key               Get flag definition
POST   /v1/flags                    Create flag (key, name, enabled, default_value, rules[])
PATCH  /v1/flags/:key               Update flag
DELETE /v1/flags/:key
```

---

## 2. Database Schema

```sql
flags (
  flag_key PK, name VARCHAR, description TEXT,
  enabled BOOLEAN DEFAULT false, default_value BOOLEAN DEFAULT false,
  rules JSON,  -- [{ "type": "allowlist", "user_ids": ["u1"] }, { "type": "percentage", "percentage": 20 }]
  version INT DEFAULT 1, created_at, updated_at
);

audit_log (id PK, flag_key FK, action VARCHAR, old_value JSON, new_value JSON, actor_id, created_at);
```

---

## 3. Key Classes / Modules

```text
FlagService
  - evaluate(flagKey, context) → boolean
  - evaluateAll(flagKeys[], context) → map[flagKey]boolean
  - getFlag(flagKey) → flag

Evaluator
  - evaluate(flag, context) → boolean
  - applyRules(flag.rules, context) in order; return first match or flag.default_value

RuleEvaluators
  - allowlist(rule, context): context.user_id in rule.user_ids
  - segment(rule, context): for each (attr, values) in rule.attrs: context[attr] in values
  - percentage(rule, context): hash(context.user_id + flag_key) % 100 < rule.percentage

FlagRepository
  - getByKey(flagKey) → flag
  - list() → flags[]
  - save(flag)  // create/update
  - invalidateCache(flagKey)

Cache (Redis)
  - get("flag:" + key) → flag JSON
  - set("flag:" + key, flag, ttl)
  - pubsub: publish "flag:invalidated" key on update
```

---

## 4. Evaluation (Steps)

1. flag = Cache.get("flag:" + key); if miss, FlagRepository.getByKey(key); Cache.set(flag).
2. If !flag.enabled return flag.default_value.
3. For rule in flag.rules:
   - if rule.type == "allowlist" and context.user_id in rule.user_ids: return true.
   - if rule.type == "segment" and matchAttrs(rule.attrs, context): return rule.value.
   - if rule.type == "percentage" and hash(context.user_id + flag_key) % 100 < rule.percentage: return true.
4. return flag.default_value.

---

## 5. Hash for Percentage (Stable)

- Use deterministic hash: e.g. MD5(user_id + ":" + flag_key) or MurmurHash; take first 8 bytes as uint64; value = value % 100. Same user + flag always same bucket.
- Salt with flag_key so different flags get different buckets per user.

---

## 6. Cache Invalidation

- On PATCH/DELETE flag: update DB; Redis PUBLISH "flag:invalidated" flag_key; all app instances subscribe and Cache.del("flag:" + flag_key).
- Or: set short TTL (e.g. 60s) and accept eventual consistency.

---

## 7. Batch Evaluate

- evaluateAll(keys, context): load all flags (batch get from cache or DB); for each key run Evaluator.evaluate(flag, context); return map. Reduces round-trips when app checks many flags per request.

---

## 8. Error Handling

- Unknown flag key: return default_value false (or 404 in admin).
- Invalid rules JSON: 400 on admin save.
- Cache/DB down: fallback to default_value or cached copy with stale warning.
