# High-Level Design: SOLID & Design Patterns in System Design

## 1. Overview

This document describes how **SOLID principles** and **design patterns** apply at a high level when designing systems and services: where they fit (API layer, service layer, data layer) and what problems they solve.

---

## System Design Process (When Applying Patterns)

When designing any system, follow: **Step 1: Clarify Requirements** (functional/non-functional). **Step 2: High-Level Design** (components, data flow)—patterns help here (e.g. Strategy, Observer). **Step 3: Detailed Design** (DB, API)—patterns in API/service layer (Factory, State). **Step 4: Scale & Optimize** (LB, sharding, cache). Flow diagrams and API lists are in each topic’s HLD/LLD (e.g. 39–59).

---

## 2. Where Patterns Apply (HLD View)

| Layer | Examples |
|-------|----------|
| **API / Gateway** | Strategy (routing, auth), Chain of Responsibility (middleware), Adapter (third-party API), Proxy (cache, rate limit) |
| **Service / Domain** | Factory, Abstract Factory (create entities), State (order/booking lifecycle), Strategy (pricing, matching), Observer (events, notify-me) |
| **Data / Persistence** | Repository (abstraction), Unit of Work (transaction), Singleton (connection pool) |
| **Cross-cutting** | Decorator (logging, retry), Facade (orchestration), Builder (complex config) |

---

## 3. Requirements (What We Achieve)

- **Single Responsibility:** Each service/class has one reason to change; e.g. OrderService (order lifecycle) vs PaymentService (charge).
- **Open/Closed:** Extend via new implementations (e.g. new payment type) without changing existing code.
- **Liskov Substitution:** Implementations of an interface can replace each other without breaking callers.
- **Interface Segregation:** Small, focused interfaces (e.g. Notifiable, Payable) instead of one fat contract.
- **Dependency Inversion:** High-level modules depend on abstractions (e.g. IInventoryService); concrete implementations injected.

---

## 4. High-Level Architecture (Patterns in Context)

```
┌─────────────────────────────────────────────────────────────────┐
│  API Layer                                                        │
│  Chain of Responsibility: Auth → RateLimit → Validate → Handler   │
│  Adapter: External API → Our IPayment interface                   │
│  Proxy: CacheProxy for read-heavy endpoints                      │
└─────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────┐
│  Service Layer                                                    │
│  Strategy: PricingStrategy, MatchingStrategy                      │
│  State: OrderState (Pending → Paid → Shipped)                     │
│  Observer: NotifyMe (Product = Subject, Users = Observers)       │
│  Factory: VehicleFactory, PaymentFactory                          │
└─────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────┐
│  Data / Components                                               │
│  Composite: FileSystem (File + Directory)                         │
│  Singleton: Config, ConnectionPool                                │
│  Builder: QueryBuilder, RequestBuilder                            │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Key Design Decisions

- **State machines** (order, booking, elevator) → State pattern for clear transitions and behavior per state.
- **Pluggable algorithms** (pricing, matching, routing) → Strategy pattern.
- **Event-driven notifications** (notify-me, pub/sub) → Observer pattern.
- **Stepwise construction** (pizza, complex config) → Builder pattern.
- **Tree structures** (file system, UI) → Composite pattern.
- **Conditional pipeline** (validation, logging handlers) → Chain of Responsibility.

---

## 6. Trade-offs

| Approach | Benefit | Cost |
|----------|---------|------|
| Patterns everywhere | Extensibility, testability | More classes, indirection |
| Minimal patterns | Simplicity | Harder to extend later |
| **Recommendation** | Use patterns where variation is expected (payment, pricing, state); keep rest simple. | — |

---

## 7. Interview Steps

1. **Clarify:** Is the question about OOP design (LLD) or system components (HLD)? Patterns apply to both.
2. **HLD:** Identify components (services, queues, stores); assign responsibilities (SRP); define interfaces between them (DIP).
3. **LLD:** Within a component, choose patterns (State for lifecycle, Strategy for algorithms, Observer for notify).
4. **Mention:** SOLID when justifying single service responsibility, interfaces, and dependency injection.

---

## Interview-Readiness Enhancements

### Capacity & SLO framing
- Define read/write QPS separately and estimate peak vs average traffic.
- Add latency budgets (p95/p99) per critical hop and target availability.
- State durability target and expected data growth/day.

### Critical path clarity
- Document write path (authoritative commit first, async side-effects second).
- Document read path (cache/read model first, fallback to source of truth).
- Identify likely hotspots (hot keys, hot partitions, fanout spikes).

### Failure handling
- Define retry strategy (bounded retries, backoff, jitter).
- Add circuit breakers and bulkheads for unstable dependencies.
- Cover queue failures (DLQ, replay) and datastore failover behavior.

### Security, operations, and cost
- Baseline security: AuthN/AuthZ, encryption in transit/at rest, secrets rotation.
- Observability: golden signals, SLO alerts, tracing, runbooks, canary/rollback.
- DR/cost: explicit RTO/RPO and top cost drivers with optimization levers.

### Trade-off table (mandatory)
- Include at least two realistic alternatives with decision rationale for this system.

