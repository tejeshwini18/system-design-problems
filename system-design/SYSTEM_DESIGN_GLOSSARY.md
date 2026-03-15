# System Design Glossary (Important Terms + Examples)

| Term | Meaning | Example |
|---|---|---|
| Availability | Percentage of time a system is operational. | 99.9% availability allows ~43.8 minutes downtime/month. |
| Latency (p50/p95/p99) | Time to serve a request at different percentiles. | API p95 = 200 ms means 95% requests finish within 200 ms. |
| Throughput | Number of requests/events processed per unit time. | 20k requests per second at peak. |
| Scalability | Ability to handle growth in load. | Add more stateless API pods behind a load balancer. |
| Elasticity | Auto-adjusting resources with changing traffic. | Scale from 10 to 100 workers during flash sale. |
| Horizontal Scaling | Adding more machines/instances. | 4 app servers increased to 20 app servers. |
| Vertical Scaling | Increasing capacity of one machine. | Move DB from 8 vCPU to 32 vCPU instance. |
| CAP Theorem | In partition, choose Consistency or Availability. | Banking prefers CP; social feed often prefers AP. |
| Consistency | How fresh/identical reads are after writes. | Read-after-write consistency in user profile updates. |
| Eventual Consistency | Data converges over time, not instantly. | Followers count may lag for a few seconds. |
| Strong Consistency | Most recent write is visible immediately. | Ledger balance after payment authorization. |
| Partition Tolerance | System keeps operating despite network splits. | Multi-AZ service continues during AZ network issue. |
| Idempotency | Same request repeated gives same result. | `POST /payments` with idempotency-key avoids double charge. |
| Caching | Storing frequent data for faster access. | Redis cache for hot product catalog data. |
| Cache Invalidation | Strategy to remove stale cache entries. | Invalidate product cache after inventory change. |
| Cache Eviction | Policy when cache is full. | LRU evicts least recently used key. |
| Load Balancer | Distributes traffic across backends. | NGINX/ALB routes HTTP requests to healthy pods. |
| Reverse Proxy | Front server that forwards requests internally. | API Gateway as reverse proxy to microservices. |
| API Gateway | Entry point for auth, rate limits, routing. | One public endpoint fans to many services. |
| Rate Limiting | Restricting request volume per identity. | 100 req/min per user token. |
| Circuit Breaker | Stops cascading failures to unhealthy dependency. | Open circuit when payment provider has repeated timeouts. |
| Retry with Backoff | Retrying failed calls with increasing delay. | Exponential retry at 100ms, 300ms, 900ms. |
| Dead Letter Queue (DLQ) | Stores messages that repeatedly fail processing. | Failed notification events go to DLQ for inspection. |
| Message Queue | Buffers asynchronous work items. | Order placed event queued for email notification. |
| Event Streaming | Durable ordered event log for consumers. | Kafka topic for clickstream events. |
| Pub/Sub | Publishers emit events; subscribers consume them. | Inventory update published to search + recommendation services. |
| Sharding | Partitioning data across multiple databases. | Users split by `user_id % N` shard key. |
| Replication | Copying data to multiple nodes. | Primary-replica DB for read scaling. |
| Read Replica | Replica serving read-only queries. | Analytics reads from replicas, writes to primary. |
| Leader Election | Choosing one node for coordination duties. | One scheduler leader assigns jobs. |
| Consistent Hashing | Evenly maps keys to nodes with minimal remap. | Redis cluster slot movement on node add/remove. |
| CDN | Edge cache near users for static/media content. | Images served from nearest PoP. |
| Object Storage | Blob storage for large unstructured data. | S3 bucket for videos and backups. |
| RPO | Max acceptable data loss window. | RPO = 5 min means backups/replication within 5 minutes. |
| RTO | Max acceptable recovery time. | RTO = 30 min for regional failover. |
| SLO | Internal target for service quality. | p95 latency < 250 ms for 99% of days. |
| SLA | External contractual commitment. | 99.95% uptime promised to enterprise customers. |
| Error Budget | Allowable unreliability derived from SLO. | 99.9% SLO gives 0.1% monthly error budget. |
| Observability | Ability to understand system internals from outputs. | Metrics + logs + traces in one dashboard. |
| Distributed Tracing | End-to-end request visibility across services. | Trace payment call from API to gateway to bank adapter. |
| Data Partition Key | Field used to route data to shard/partition. | `tenant_id` as partition key in multi-tenant SaaS. |
| Hot Partition | Unevenly loaded partition due to skewed keys. | Celebrity user causing one timeline shard overload. |
| CQRS | Separate write model from read model. | Orders written in OLTP DB, queried via denormalized read store. |
| Saga Pattern | Distributed transaction via local transactions + compensations. | Cancel inventory reservation if payment fails. |
| Two-Phase Commit (2PC) | Coordinator-driven distributed commit protocol. | Used when strict atomicity needed across services. |
| Optimistic Locking | Detect conflicts using version checks. | Update row only if `version` matches prior read. |
| Pessimistic Locking | Lock resource before update to avoid conflict. | Seat lock in ticket booking checkout. |
| Bloom Filter | Probabilistic set-membership check (with false positives). | Quickly reject non-existent short URLs before DB hit. |
| Quorum (R/W) | Minimum replica acknowledgments for read/write. | Cassandra with `W=2, R=2, N=3`. |
| Backpressure | Slowing producers when consumers are overloaded. | Kafka consumer lag triggers producer throttling. |
| Multi-Tenancy | Single system serves multiple customer tenants. | Tenant-isolated data + rate limits per tenant. |
