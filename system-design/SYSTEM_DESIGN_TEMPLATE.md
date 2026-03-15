# System Design Document Template

Every design in this folder follows this process and structure.

---

## System Design Process

### Step 1: Clarify Requirements
- **Functional:** What the system must do (features, use cases).
- **Non-functional:** Latency, availability, scale, consistency, durability.
- **Constraints & assumptions:** Tech stack, existing systems, scope.
- **Success metrics:** User-facing and platform KPIs for v1.

### Step 2: High-Level Design
- **Components:** Main services/stores and their responsibilities.
- **Interfaces:** How components talk (REST/gRPC/events).
- **Data flow:** Step-by-step for read + write critical paths.
- **Trust boundaries:** Public edge, internal network, data/security boundaries.
- **Flow diagram:** Architecture and/or sequence (Mermaid + Eraser).

### Step 3: Detailed Design
- **Database:** SQL vs NoSQL choice; schema or key model.
- **API design:** Full list of endpoints (method, path, purpose, idempotency).
- **Key algorithms:** Critical logic (e.g., ranking, allocation, dedupe).
- **Concurrency:** optimistic/pessimistic locking, ordering, retries, idempotent consumers.

### Step 4: Scale & Optimize
- **Load balancing:** How traffic is distributed.
- **Partitioning/sharding:** How data and workload are distributed.
- **Caching:** What is cached, where, invalidation.
- **Asynchronous processing:** queues/streams, replay, DLQ, backpressure.

---

## Mandatory Interview-Ready Additions

1. **Back-of-envelope estimation**
   - Peak reads/writes QPS, data growth/day, bandwidth, fanout estimates.
2. **SLO/SLA framing**
   - p95/p99 latency targets, availability target, error budget implications.
3. **Failure-mode analysis**
   - Dependency outage, datastore outage, queue lag/replay, degradation strategy.
4. **Security/privacy baseline**
   - AuthN/AuthZ, encryption at rest/in transit, secrets, audit trail, abuse/rate limiting.
5. **Observability/operations**
   - Golden signals, business KPIs, tracing, alerts, runbooks, canary + rollback.
6. **DR and compliance**
   - RTO/RPO, backup/restore, retention policy, region failover posture.
7. **Cost analysis**
   - Major cost drivers and optimization levers.
8. **Trade-offs table**
   - At least 2 alternatives with decision rationale.

---

## Document Structure

### HLD.md
1. Title & overview
2. Problem framing + assumptions + success metrics
3. Requirements (functional/non-functional/constraints)
4. Back-of-envelope capacity estimate
5. High-Level Architecture (Mermaid component diagram + optional Eraser)
6. Critical flows (read path + write path)
7. API surface overview
8. Data model choices + storage strategy
9. Scalability strategy (x10/x100 plan)
10. Reliability and failure handling
11. Security/privacy/compliance
12. Observability + SLOs + alerts
13. Cost considerations
14. Trade-offs and alternatives
15. Future improvements

### LLD.md
1. Scope and assumptions
2. Full API endpoints (method/path/request/response/errors/idempotency)
3. Detailed schema / key model / indexes / partition key
4. Class/module design and responsibilities
5. Core algorithms and pseudocode
6. Concurrency, ordering, and consistency behavior
7. Failure scenarios and compensation logic
8. Test strategy (unit/integration/load/failure tests)
9. Operational concerns (rollout, migration, monitoring)

---

## Diagrams

- **Mermaid:** Renders in GitHub, VS Code, many IDEs. Use `flowchart`, `sequenceDiagram`, etc.
- **Eraser:** Paste the code block into [Eraser.io](https://app.eraser.io) → New diagram → Diagram as code.

Minimum expectation per design:
- HLD: 1 component diagram + 1 sequence/flow diagram.
- LLD: at least 1 detailed sequence/flow for a critical path.
