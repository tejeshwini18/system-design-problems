# Interview Transcript — User Authentication System

When I explain this in an interview, I first define scope: this is a centralized authentication platform for signup, login, logout, token refresh, password reset, and optional MFA. I clarify that I will separate identity proofing from authorization and that my design should support web, mobile, and service-to-service access patterns.

Then I describe the critical flows end-to-end. For signup, I create a user in pending state and send asynchronous verification. For login, I validate credentials, enforce abuse checks, optionally trigger MFA, and issue short-lived access plus long-lived refresh tokens. For refresh, I use rotation with one-time token usage to reduce replay risk. For logout or password reset, I revoke active sessions to contain compromise.

After that, I explain storage and APIs. The user table is authoritative for credentials and account state, while a session store tracks refresh token JTIs per device. I call out indexes and transactional boundaries, especially around refresh token rotation and one-time reset/verification tokens. I also explain consistency: strong consistency for credential/session state and eventual consistency for analytics and reporting.

Next, I cover security and reliability as first-class concerns. I mention Argon2id hashing, key management and rotation, TLS everywhere, structured audit logging, and anomaly detection signals. On the resilience side, I add retries with jitter for email/SMS providers, circuit breakers for third-party identity providers, and DLQ replay for failed notifications.

Finally, I close with trade-offs and roadmap. I justify JWT access tokens for low-latency verification while keeping refresh tokens stateful for revocation control. If traffic grows 10x, I would shard session storage, add regional auth edges with global key distribution, and expand risk-based authentication and passkey support.
