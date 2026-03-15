# Interview-Ready Transcripts for System Design Problems (Improved)

These transcripts are **problem-specific** and incorporate modern interview expectations: capacity assumptions, SLOs, read/write path clarity, failure handling, security/compliance, observability, and trade-offs.

Use this format in interviews: **Clarify → Estimate → HLD → LLD → Reliability/Security → Trade-offs → Roadmap**.

## 1. Scalable Chat Application

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Scalable Chat Application** (Messaging).
- **v1 functional scope:** messages, conversations, presence, delivery receipts.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** WebSocket gateway, chat service, presence service, message store, fanout queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, conversations, messages.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **push vs pull delivery; ordered delivery vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 2. URL Shortener

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **URL Shortener** (Redirection).
- **v1 functional scope:** shorten URL, redirect, analytics.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api service, id-generator, redirect cache, url store, click stream.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** urls, redirects, clicks.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **random key vs hash key; cache hit ratio vs cost**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 3. API Rate Limiter

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **API Rate Limiter** (Protection).
- **v1 functional scope:** enforce per-key quotas, burst control.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** gateway plugin, policy store, redis counter, config service.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** limits, counters, tokens.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **token bucket vs sliding window**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 4. Distributed Cache System

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Distributed Cache System** (Acceleration).
- **v1 functional scope:** get/set/delete with ttl and eviction.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** cache router, shard manager, storage nodes, gossip/health.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** cache entries, metadata.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **write-through vs write-back**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 5. Notification System

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Notification System** (Fanout).
- **v1 functional scope:** multi-channel notifications with retries/preferences.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** orchestrator, channel workers, provider adapters, dlq.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** templates, notification jobs, delivery status.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **sync send vs async queue**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 6. Ride-Hailing System

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Ride-Hailing System** (Matching).
- **v1 functional scope:** rider requests, driver matching, trip lifecycle.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** geo-index service, matching engine, trip service, pricing.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** riders, drivers, trips, locations.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **nearest-first vs surge-aware match quality**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 7. Video Streaming Platform

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Video Streaming Platform** (Media).
- **v1 functional scope:** upload/transcode/stream/recommend.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** upload api, transcoder, object storage, cdn, metadata db.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** videos, renditions, watch sessions.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **precompute renditions vs just-in-time**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 8. Payment Gateway

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Payment Gateway** (Transactions).
- **v1 functional scope:** authorize/capture/refund with idempotency.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** payment api, risk engine, ledger db, provider adapters.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** payments, ledger entries, payouts.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **strong consistency vs availability during provider outage**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 9. News Feed System

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **News Feed System** (Timeline).
- **v1 functional scope:** post, follow, timeline read.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** write service, fanout workers, timeline store, ranking.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, posts, edges, timelines.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **fanout-on-write vs fanout-on-read**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 10. Search Autocomplete

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Search Autocomplete** (Search).
- **v1 functional scope:** prefix suggestions and ranking.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** ingestion pipeline, trie/index, ranking service, cache.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** queries, terms, popularity.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **freshness vs query latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 11. Logging & Monitoring

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Logging & Monitoring** (Observability).
- **v1 functional scope:** collect, index, search logs.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** agents, ingest pipeline, indexer, storage tiers.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** logs, indices, retention policies.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **hot retention vs storage cost**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 12. File Storage System

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **File Storage System** (Storage).
- **v1 functional scope:** upload/download/share/version files.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** metadata service, chunk store, object storage, cdn.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** files, chunks, permissions.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **large chunk size vs parallelism**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 13. E-commerce Checkout

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **E-commerce Checkout** (Commerce).
- **v1 functional scope:** cart, checkout, payment, order state.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** checkout service, inventory service, payment service, order saga.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** carts, orders, inventory.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **sync inventory lock vs optimistic reservation**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 14. Leaderboard System

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Leaderboard System** (Ranking).
- **v1 functional scope:** score updates and top-N queries.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** score writer, sorted-set store, query api.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** players, scores, seasons.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **exact rank vs approximation cost**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 15. Distributed Job Scheduler

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Distributed Job Scheduler** (Scheduling).
- **v1 functional scope:** schedule, retry, execute distributed jobs.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** scheduler, worker pool, lease manager, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** jobs, triggers, executions.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **at-least-once vs exactly-once**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 16. Feature Flag System

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Feature Flag System** (Experimentation).
- **v1 functional scope:** flag rollout by user/segment.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** flag control plane, sdk cache, rule engine.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** flags, rules, evaluations.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **server-eval vs client-eval**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 17. Metrics & Analytics Platform

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Metrics & Analytics Platform** (Analytics).
- **v1 functional scope:** ingest time-series metrics and aggregate.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** collector, stream processor, tsdb, query api.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** metrics, tags, rollups.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **high-cardinality support vs storage cost**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 18. YouTube: 2.49B Users With MySQL

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **YouTube: 2.49B Users With MySQL** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 19. Uber: Nearby Drivers at 1M RPS

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Uber: Nearby Drivers at 1M RPS** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 20. AWS: Scale App to 10M Users

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **AWS: Scale App to 10M Users** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 21. Meta: 11.5M Serverless Calls/s

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Meta: 11.5M Serverless Calls/s** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 22. Google Docs: Real-Time Collaboration

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Google Docs: Real-Time Collaboration** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 23. Bluesky: Decentralized Social

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Bluesky: Decentralized Social** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 24. Slack: Real-Time Messaging

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Slack: Real-Time Messaging** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 25. Spotify: Music Streaming

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Spotify: Music Streaming** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 26. ChatGPT: How LLMs Work

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **ChatGPT: How LLMs Work** (Messaging).
- **v1 functional scope:** messages, conversations, presence, delivery receipts.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** WebSocket gateway, chat service, presence service, message store, fanout queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, conversations, messages.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **push vs pull delivery; ordered delivery vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 27. Twitter: Timeline

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Twitter: Timeline** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 28. Reddit: Communities & Voting

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Reddit: Communities & Voting** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 29. Apache Kafka: Distributed Streaming

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Apache Kafka: Distributed Streaming** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 30. Distributed ID Generator

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Distributed ID Generator** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 31. Load Balancer

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Load Balancer** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 32. Design Instagram

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Design Instagram** (Social media).
- **v1 functional scope:** post media, follow graph, feed.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** media pipeline, feed service, graph store, cdn.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, media, follows.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **feed freshness vs compute cost**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 33. Search Engine

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Search Engine** (Web search).
- **v1 functional scope:** crawl/index/query/rank.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** crawler, indexer, query broker, ranker.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** docs, terms, links.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **index freshness vs serving latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 34. Web Crawler

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Web Crawler** (Crawling).
- **v1 functional scope:** discover/fetch/parse pages respectfully.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** url frontier, fetchers, parser, deduper.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** urls, frontier, robots policies.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **crawl speed vs politeness**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 35. Distributed Lock

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Distributed Lock** (Coordination).
- **v1 functional scope:** acquire/release/renew locks safely.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** lock service, consensus store, watchdog.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** locks, owners, leases.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **safety vs lock acquisition latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 36. CDN

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **CDN** (Edge delivery).
- **v1 functional scope:** cache and serve static content globally.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** edge pops, origin shield, invalidation bus.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** assets, cache keys, invalidations.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **high ttl hit-rate vs freshness**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 37. Database Sharding

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Database Sharding** (Data scale).
- **v1 functional scope:** partition and rebalance data.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** router, shard map manager, migration worker.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** shards, ranges, mappings.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **range vs hash sharding**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 38. SOLID & Design Patterns

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **SOLID & Design Patterns** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 39. Notify-Me Button

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Notify-Me Button** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 40. Pizza Billing System

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Pizza Billing System** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 41. Parking Lot

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Parking Lot** (LLD object model).
- **v1 functional scope:** slot allocation and billing rules.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** entry/exit controllers, allocator, pricing engine.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** vehicles, slots, tickets.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **simple allocation vs optimal packing**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 42. Snake & Ladder

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Snake & Ladder** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 43. Elevator System

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Elevator System** (LLD object model).
- **v1 functional scope:** request scheduling and state transitions.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** dispatcher, state machine, car controllers.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** elevators, requests, floors.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **throughput vs wait-time fairness**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 44. Car Rental

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Car Rental** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 45. Logging System (LLD)

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Logging System (LLD)** (Observability).
- **v1 functional scope:** collect, index, search logs.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** agents, ingest pipeline, indexer, storage tiers.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** logs, indices, retention policies.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **hot retention vs storage cost**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 46. Tic-Tac-Toe

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Tic-Tac-Toe** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 47. Chess Game

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Chess Game** (LLD object model).
- **v1 functional scope:** move validation and game state.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** game engine, rule validator, history manager.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** board, pieces, moves.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **strict rules engine complexity vs extensibility**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 48. File System

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **File System** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 49. Splitwise & Simplify Algorithm

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Splitwise & Simplify Algorithm** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 50. Vending Machine

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Vending Machine** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 51. ATM

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **ATM** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 52. BookMyShow & Concurrency

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **BookMyShow & Concurrency** (Ticketing).
- **v1 functional scope:** seat search, lock, payment, booking.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** catalog, seat-lock service, booking saga.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** shows, seats, holds, bookings.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **lock duration vs conversion rate**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 53. Library Management System

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Library Management System** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 54. Traffic Light System

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Traffic Light System** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 55. Meeting Scheduler

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Meeting Scheduler** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 56. Online Voting System

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Online Voting System** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 57. Inventory Management System

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Inventory Management System** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 58. Restaurant Management System

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Restaurant Management System** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 59. Bowling Alley Machine

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Bowling Alley Machine** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 60. Pastebin

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Pastebin** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 61. Ticketmaster

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Ticketmaster** (General system).
- **v1 functional scope:** core create/read/update workflows.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** api gateway, core service, cache, db, queue.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** users, resources, events.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs latency**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 62. Hotel Booking

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Hotel Booking** (Reservation).
- **v1 functional scope:** search rooms, reserve, pay.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** inventory service, pricing, reservation service.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** hotels, rooms, bookings.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **overbooking risk vs occupancy**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 63. Food Delivery

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **Food Delivery** (Logistics).
- **v1 functional scope:** order, dispatch, tracking.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** order service, dispatch engine, tracking service.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** restaurants, orders, couriers.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **dispatch quality vs assignment speed**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---

## 64. LMS & Calendar

**Interviewer:** Design this system.

**Candidate:** Sure—let me confirm scope for **LMS & Calendar** (Productivity).
- **v1 functional scope:** events, invites, reminders.
- **NFR targets:** p95 read < 200 ms, p95 write < 300 ms, 99.9% availability (raise to 99.95% for critical paths), and multi-AZ durability.
- **Scale assumption (starter):** ~1M DAU, 10x peak-to-average traffic, and growth-ready partitioning strategy.

**Candidate (Back-of-envelope):**
- Estimate peak QPS for read/write separately, then size cache and DB capacity from that split.
- Estimate daily storage growth and retention policy (hot/warm/cold tiers).
- Define latency budget per hop (gateway, service, cache/DB, downstream).

**Candidate (HLD):**
- **Core components:** calendar api, sync worker, notification service.
- **Write path:** validate/authenticate → persist authoritative state → publish event for async side-effects.
- **Read path:** serve from cache/read model first, fallback to primary store with controlled fanout.
- **Bottleneck watchpoints:** hot keys/partitions, queue lag, cross-region latency, and fanout spikes.

**Candidate (LLD deep dive):**
- **Key entities/data model:** calendars, events, attendees.
- **API contracts:** include idempotency keys on non-idempotent write operations; use pagination/cursors for list endpoints.
- **Concurrency strategy:** optimistic locking by default; short-lived pessimistic locks only for scarce resources.
- **Consistency choice:** strong consistency only where correctness is user-visible/financially critical; eventual consistency elsewhere.

**Candidate (Reliability & Failure Modes):**
- Dependency timeout/retry (exponential backoff + jitter), circuit breakers, and per-dependency bulkheads.
- Queue failures handled with DLQ + replay; duplicate-event safety via idempotent consumers.
- Data-store failures handled with replica failover and graceful degradation for non-critical features.

**Candidate (Security/Compliance):**
- AuthN/AuthZ, tenant isolation, encryption in transit/at rest, and secret rotation.
- PII minimization + audit logging for sensitive mutations; rate limiting + abuse controls.
- For payment/regulated flows: idempotency, immutable ledger/audit trail, and policy-based retention.

**Candidate (Observability/Operations):**
- Golden signals (latency, traffic, errors, saturation), business KPIs, and distributed traces.
- SLOs with alerting tied to error budget burn; runbooks and canary/rollback playbook.
- DR stance: explicit RTO/RPO targets and periodic failover drills.

**Candidate (Trade-offs):**
- Primary trade-off: **consistency vs offline availability**.
- Also evaluate cost drivers: cache hit rate, storage tiering, and network egress/CDN usage.

**Candidate (Close):** I would ship a narrow v1, validate bottlenecks with production telemetry, then scale to x10/x100 with partitioning, async pipelines, and selective denormalization.

---
