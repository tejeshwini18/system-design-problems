# Interview Transcript — CDN

When I explain this in an interview, I start by framing the problem in plain language. I describe **CDN** as a content delivery network and confirm the v1 scope, expected user behavior, and non-functional targets such as latency, availability, and durability. Before drawing architecture, I align on realistic traffic assumptions so every decision is justified with capacity context rather than guesswork.

Next, I walk through the end-to-end flow from the user request to final response. I explain that the system first validates the request, then processes domain data like content keys, cache metadata, invalidation events, and origin mappings, and finally returns the result through stable APIs. At the architecture level, I describe how components like edge POP caches, routing/DNS layer, origin shield, and invalidation pipeline collaborate on the read and write paths, and I call out where asynchronous processing is introduced to protect latency.

After that, I shift to data and API design in human terms. I explain which entities are authoritative, where indexing or partitioning is needed, and how idempotency prevents duplicate outcomes during retries. I also clarify consistency expectations so the interviewer can see exactly where strong consistency is required and where eventual consistency is acceptable for scale.

Then I cover reliability as a production scenario, not just theory. I describe dependency failures, retries with backoff and jitter, circuit breakers, dead-letter handling, and graceful degradation so critical user journeys continue to work. I pair this with security and operations: authentication/authorization, encryption, auditability, observability signals, SLO-based alerting, and rollback/DR posture with clear RTO/RPO goals.

Finally, I close with trade-offs and roadmap. For this design, I highlight bottlenecks such as cache hit ratio vs freshness control, explain why the chosen approach is suitable for v1, and mention what I would evolve for x10 traffic. This keeps the transcript interview-friendly, technically grounded, and easy for a panel to evaluate across HLD and LLD depth.
