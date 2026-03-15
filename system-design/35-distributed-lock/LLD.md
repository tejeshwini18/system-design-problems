# Low-Level Design: Distributed Lock Service

## 1. API (Client Interface)

```text
Lock lock = lockService.acquire("order_123", 10_000)   // key, ttl_ms
if lock != null:
  try:
    doWork()
  finally:
    lock.release()
else:
  // failed to acquire
```

- **Try-lock:** acquire(key, ttl) returns immediately with success or failure.  
- **Blocking:** acquire(key, ttl, waitTimeout) polls or watches until acquired or timeout.

---

## 2. Key Classes / Modules

```text
LockService
  - acquire(key, ttlMs) → Lock | null
  - release(lock) → void

Lock
  - key: string
  - owner: string   // unique token
  - ttlMs: long
  - release()
  - refresh() → boolean   // extend TTL

RedisLockStore
  - tryAcquire(key, owner, ttlMs) → boolean   // SET key owner NX PX ttl
  - release(key, owner) → boolean   // Lua: get and del if match
  - refresh(key, owner, ttlMs) → boolean   // PEXPIRE if GET == owner
```

---

## 3. Redis Commands (Acquire)

```text
SET lock:{key} {owner} NX PX {ttl_ms}
  NX = set if not exists
  PX = expiry in ms
  Returns: OK if set, null if key exists
```

---

## 4. Release (Lua Script)

```lua
if redis.call("get", KEYS[1]) == ARGV[1] then
  return redis.call("del", KEYS[1])
else
  return 0
end
```
- KEYS[1] = lock key; ARGV[1] = owner. Prevents deleting lock held by another.

---

## 5. Owner Token

- Generate: UUID.randomUUID().toString() or clientId + "-" + System.nanoTime().
- Store in Lock object; must be sent on release and refresh so only holder can modify.

---

## 6. Refresh Loop (Optional)

```text
startRefreshThread(lock, intervalMs = ttlMs/2):
  while lock.held:
    if not store.refresh(lock.key, lock.owner, lock.ttlMs):
      lock.held = false; notify holder
    sleep(intervalMs)
```

- Stop thread on release(). If refresh fails (key deleted or changed), consider lock lost and notify application.

---

## 7. Error Handling

- **Redis down:** acquire returns null or throws; application retries or backs off.
- **Release on wrong node:** Lua ensures only owner can delete; no-op if key already gone (idempotent).
- **Clock skew (Redlock):** Use NTP; document that Redlock does not guarantee safety under arbitrary clock skew (consider fencing token for critical resources).

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

