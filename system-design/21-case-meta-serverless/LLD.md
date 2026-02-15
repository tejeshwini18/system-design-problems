# Low-Level Design: Meta-Scale Serverless (11.5M invocations/s)

## 1. APIs (Control Plane)

```http
POST   /v1/functions              Create/update function (code, runtime, memory, timeout)
GET    /v1/functions/:id          Get function config
DELETE /v1/functions/:id          Delete function
POST   /v1/functions/:id/invoke  Synchronous invoke (payload, optional request_id)
```

**Async invoke:** Client posts to queue or event bus; response via webhook or polling (e.g. GET /v1/invocations/:id/result).

---

## 2. Invocation Message (Internal)

```json
{
  "invocation_id": "uuid",
  "function_id": "fn_123",
  "version": "v2",
  "payload": { ... },
  "deadline_ms": 1707890123456,
  "memory_mb": 256,
  "request_id": "req_xxx"
}
```

- Router produces this to a per-region or per-function queue; workers consume and execute.

---

## 3. Key Classes / Modules

```text
EventRouter
  - onRequest(req): validate, extract function_id and payload; enqueue InvocationMessage or call InvocationRouter

InvocationRouter
  - route(msg) → (region, worker_id or queue)
  - selectRegion(msg): by latency or tenant config
  - selectWorker(region, function_id): from PlacementService

PlacementService
  - getPlacement(function_id, version) → (worker_id, warm_container_id) or (cold_start)
  - returnToPool(worker_id, container_id, function_id, version)
  - warmPool: map[(function_id, version)] → list of (worker_id, container_id)

Worker
  - execute(msg): load or get warm container; run function with payload; return result; return container to pool or destroy
  - coldStart(function_id, version): fetch code/config; create container; init runtime; return container_id
```

---

## 4. Warm Pool (Data Structure)

```text
Per (function_id, version):
  pool: Queue<(worker_id, container_id)>
  max_pool_size: 100
  refill_threshold: 20   // when size < 20, trigger refill

get(): pool.pop() or null
put(worker_id, container_id): pool.push() if size < max_pool_size else destroy container
refill(): for i in 1..N: coldStart() and put(); run in background
```

- Refill triggered when pool size drops below threshold; async so it doesn’t block get().

---

## 5. Worker Execution Loop

```text
while true:
  msg = queue.consume()
  placement = PlacementService.getPlacement(msg.function_id, msg.version)
  if placement.warm:
    container = placement.container
  else:
    container = coldStart(msg.function_id, msg.version)
  result = runInSandbox(container, msg.payload, msg.deadline_ms)
  sendResult(msg.invocation_id, result)
  PlacementService.returnToPool(worker_id, container, msg.function_id, msg.version)
```

---

## 6. Scaling Numbers (Back-of-Envelope)

- 11.5M invocations/s; assume 100 ms average duration → 1.15M concurrent executions.
- If each worker handles 500 concurrent: ~2.3K workers (or 23K if 50 concurrent each).
- Partitions: by function_id hash so each partition gets a subset of invocations; each partition has its own worker pool and queue.

---

## 7. Error Handling

- **Timeout:** Worker kills execution at deadline; return timeout error to client.
- **OOM:** Sandbox limit; kill and return error; do not return container to pool.
- **Worker crash:** Invocation retried by queue (visibility timeout or at-least-once); idempotent function or client retry for exactly-once semantics.
- **Cold start failure:** Retry on another worker; alert if repeated.
