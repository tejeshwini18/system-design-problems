# Interview Transcript — Chess Game

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Chess Game** (LLD object model).
- **v1 functional scope:** move validation and game state.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (99.95% for critical paths), multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, growth-ready partitioning.

## Back-of-envelope
- Estimate peak read/write QPS separately; derive cache/DB sizing from split.
- Estimate storage growth/day and hot/warm/cold retention strategy.
- Define end-to-end latency budget per hop.

## HLD Walkthrough
- **Core components:** game engine, rule validator, history manager.
- **Write path:** validate/authenticate → persist source of truth → publish async events.
- **Read path:** serve from cache/read model → fallback to primary store.
- **Hotspots:** hot partitions, queue lag, cross-region latency, fanout bursts.

## LLD Deep Dive
- **Key entities/data model:** board, pieces, moves.
- **API contracts:** idempotency keys for non-idempotent writes; cursor pagination for reads.
- **Concurrency:** optimistic locking by default; short-lived pessimistic locks for scarce resources.
- **Consistency:** strong where correctness is critical; eventual where latency/scale dominates.

## Reliability, Security, and Operations
- Retries with backoff+jitter, circuit breaker, bulkheads; DLQ + replay for async failures.
- AuthN/AuthZ, encryption in transit/at rest, secret rotation, audit logging for sensitive actions.
- Golden signals + business KPIs + traces; SLO alerts; canary/rollback; DR with RTO/RPO targets.

## Trade-offs and Close
- **Primary trade-off:** strict rules engine complexity vs extensibility.
- **Cost focus:** cache hit rate, storage tiering, and network egress/CDN costs.
- **Close:** ship narrow v1, validate bottlenecks with telemetry, then scale x10/x100.
