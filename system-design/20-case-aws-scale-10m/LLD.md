# Low-Level Design: Scaling to 10M Users on AWS

## 1. What to Implement at Each Layer

### Application (stateless)
- No local session storage; session id in cookie/token; session data in Redis.
- Health check endpoint (e.g. `/health`) for ALB: return 200 when app and Redis (and optionally DB) are reachable.
- Use RDS read endpoint for SELECTs and primary endpoint for writes (or use a library that routes by operation).
- Cache key design: e.g. `cache:user:{id}`, `cache:product:{id}`; TTL 5–15 min; invalidate on update.

### Load balancer
- ALB: round-robin or least outstanding requests; stickiness optional for WebSocket or temporary affinity.
- Listener: HTTPS 443 → target group (EC2 or ECS); health check interval 30s, healthy threshold 2, unhealthy 3.
- Target group: port 8080 (app); deregistration delay for graceful shutdown.

### Auto Scaling
- Min: 2 (for HA); max: 50 (or based on budget); desired: start 4.
- Scale-out: CPU &gt; 70% or RequestCountPerTarget &gt; 1000 for 2 min.
- Scale-in: CPU &lt; 30% for 5 min; scale-in policy slower than scale-out to avoid flapping.
- Cooldown or target tracking to avoid thrashing.

### Database
- Primary: single writer; automated backups; Multi-AZ standby.
- Read replicas: 1–2 for read scaling; replication lag &lt; 1 s target; application uses read endpoint for read-only queries.
- Connection pooling: RDS Proxy or PgBouncer so app doesn’t exhaust connections.
- Schema and indexes: index by user_id, created_at for common queries; avoid N+1.

### Cache (Redis)
- Key namespace: `app:session:{sid}`, `app:user:{id}`, `app:feed:{user_id}:page:{n}`.
- Session TTL: 7 days; refresh on activity.
- Cache-aside: on miss load from DB and set in Redis; on update delete key or set new value.

### Async (SQS)
- Queue: e.g. `email-notifications`; visibility timeout 60s; dead-letter queue after 3 receives.
- Producer: after creating order, send message `{ type: "order_created", order_id, user_id }`.
- Consumer: Lambda or EC2 worker; process message; delete on success; on failure let visibility expire for retry.

---

## 2. Configuration Checklist (AWS)

| Resource | Key settings |
|----------|----------------|
| ALB | HTTPS listener; cert in ACM; target group = app instances |
| ASG | Launch template (AMI, instance type, user data); min/max/desired; scaling policies |
| RDS | Multi-AZ; read replica(s); parameter group (connections, timeouts) |
| ElastiCache | Cluster mode or single node; eviction policy; in VPC |
| CloudFront | Origin = ALB or S3; cache policy; HTTPS |
| SQS | Standard or FIFO; DLQ; IAM for producers/consumers |

---

## 3. Observability

- **Logs:** App → CloudWatch Logs (or stdout and container log driver); log group per service.
- **Metrics:** CloudWatch custom metrics (e.g. requests/s, error rate); RDS and Redis metrics from AWS.
- **Alarms:** CPU &gt; 80%, DB connections &gt; 80% of max, 5xx rate &gt; 1%.
- **Tracing:** X-Ray SDK in app; trace requests across ALB → app → DB/cache.

---

## 4. Rollout and Failover

- **Deploy:** New ASG launch template version; rolling update (batch size 1 or 2); health check ensures new instances get traffic only when ready.
- **DB failover:** RDS Multi-AZ automatic failover; app reconnects to new primary (same endpoint); brief write unavailability.
- **Cache:** Redis cluster or replication; app reconnects on connection loss; optional lazy load on cold start.

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

