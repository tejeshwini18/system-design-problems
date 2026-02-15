# Low-Level Design: Load Balancer

## 1. Configuration (Representative)

```text
listeners:
  - port: 443
    protocol: https
    default_pool: api-pool
    rules:
      - host: api.example.com  → pool: api-pool
      - path: /static/*        → pool: static-pool

pools:
  api-pool:
    backends: [ 10.0.1.1:8080, 10.0.1.2:8080 ]
    health_check: { path: /health, interval: 10s, unhealthy_threshold: 3 }
    algorithm: least_connections
    sticky: cookie name=lb_session max_age=3600
  static-pool:
    backends: [ 10.0.2.1:80 ]
    algorithm: round_robin
```

---

## 2. Key Classes / Modules

```text
LoadBalancer
  - addListener(port, protocol, defaultPool, rules?)
  - handleConnection(conn): pick backend; proxy(conn, backend)

Pool
  - backends: List<Backend>
  - healthyBackends(): filter by health
  - scheduler: Scheduler
  - pickBackend(request?) → backend

Scheduler (interface)
  - pick(pool, request) → backend
  - RoundRobinScheduler: next = (next + 1) % n
  - LeastConnectionsScheduler: argmin backend.activeConnections
  - WeightedRoundRobinScheduler: by weight

HealthChecker
  - run(): for each backend: probe(); update backend.healthy
  - probe(backend): TCP connect or HTTP GET; success/fail

StickyStore
  - get(key) → backend | null
  - set(key, backend, ttl)
  - key = client_ip or cookie value
```

---

## 3. Backend Selection (Least Connections)

```text
pickBackend():
  candidates = pool.healthyBackends()
  if candidates.empty(): return 503
  return candidates.minBy(b => b.activeConnections)
```

- **Round-robin:** index = (atomic_counter++) % len(candidates); return candidates[index].

---

## 4. Sticky Session (Cookie)

- **First request:** No cookie; pick backend; set response header Set-Cookie: lb_session=<backend_id>; Max-Age=3600.
- **Subsequent request:** Read cookie lb_session; lookup backend by id; if found and healthy, use it; else pick new backend and overwrite cookie.
- **Storage:** In-memory map (backend_id → backend); or encode backend in cookie (signed) so no server state.

---

## 5. Health Check Loop

```text
every interval (e.g. 10s):
  for backend in pool.backends:
    ok = probe(backend)   // TCP or HTTP
    if ok: backend.consecutiveFailures = 0; backend.healthy = true (after threshold)
    else: backend.consecutiveFailures++; if >= 3: backend.healthy = false
```

---

## 6. Proxy (L7) Flow

1. Read HTTP request (method, path, headers, body).
2. Add X-Forwarded-For: client_ip; X-Forwarded-Proto: https.
3. Establish connection to backend (or reuse from pool); send request; stream response.
4. On backend error or timeout: retry next backend (optional) or return 502/504.
5. Increment backend.activeConnections on connect; decrement on close.

---

## 7. Error Handling

- **All backends unhealthy:** Return 503 Service Unavailable.
- **Backend timeout:** Close connection; return 504; mark backend for health recheck.
- **Connection refused:** Decrement health; next request may try another backend.
