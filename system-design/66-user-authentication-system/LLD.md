# LLD: User Authentication System

## APIs

```http
POST   /v1/auth/signup
Body: { email, password, name }
Resp: { user_id, verification_sent: true }

POST   /v1/auth/verify-email
Body: { token }
Resp: { verified: true }

POST   /v1/auth/login
Body: { email, password, device_id }
Resp: { access_token, refresh_token, expires_in }

POST   /v1/auth/mfa/challenge
Body: { login_txn_id, method }
Resp: { challenge_id, expires_in }

POST   /v1/auth/mfa/verify
Body: { challenge_id, otp }
Resp: { access_token, refresh_token, expires_in }

POST   /v1/auth/refresh
Body: { refresh_token }
Resp: { access_token, refresh_token }

POST   /v1/auth/logout
Body: { refresh_token }
Resp: { success: true }

POST   /v1/auth/logout-all
Body: { user_id }
Resp: { revoked_sessions: n }

POST   /v1/auth/forgot-password
Body: { email }
Resp: { reset_sent: true }

POST   /v1/auth/reset-password
Body: { token, new_password }
Resp: { password_reset: true }

GET    /.well-known/jwks.json
Resp: { keys: [...] }
```

## Key Modules / Classes

```text
AuthController
  - signup(req), login(req), refresh(req), logout(req), resetPassword(req)

CredentialService
  - hashPassword(pw), verifyPassword(pw, hash), needsRehash(hash)

TokenService
  - issueAccessToken(user, scopes, kid)
  - issueRefreshToken(session)
  - verifyAccessToken(token)
  - rotateRefreshToken(oldToken)

SessionService
  - createSession(userId, device)
  - revokeSession(refreshJti)
  - revokeAllSessions(userId)

RiskEngine
  - scoreLogin(ip, device, geo, historicalSignals)
  - requireMFA(score)

RateLimitService
  - check(ip, account), incrementFailure(account), resetFailure(account)
```

## SQL Schema (Example)

```sql
CREATE TABLE users (
  user_id BIGSERIAL PRIMARY KEY,
  email VARCHAR(320) UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  password_algo VARCHAR(16) NOT NULL DEFAULT 'argon2id',
  email_verified BOOLEAN NOT NULL DEFAULT false,
  mfa_enabled BOOLEAN NOT NULL DEFAULT false,
  status VARCHAR(20) NOT NULL DEFAULT 'ACTIVE',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE auth_sessions (
  session_id BIGSERIAL PRIMARY KEY,
  user_id BIGINT NOT NULL REFERENCES users(user_id),
  refresh_jti UUID NOT NULL UNIQUE,
  device_id VARCHAR(128),
  ip INET,
  user_agent TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  expires_at TIMESTAMPTZ NOT NULL,
  revoked_at TIMESTAMPTZ,
  rotated_from_jti UUID
);
CREATE INDEX idx_auth_sessions_user ON auth_sessions(user_id);
CREATE INDEX idx_auth_sessions_exp ON auth_sessions(expires_at);

CREATE TABLE password_resets (
  token_hash CHAR(64) PRIMARY KEY,
  user_id BIGINT NOT NULL REFERENCES users(user_id),
  expires_at TIMESTAMPTZ NOT NULL,
  used_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE audit_auth_events (
  event_id BIGSERIAL PRIMARY KEY,
  user_id BIGINT,
  event_type VARCHAR(50) NOT NULL,
  ip INET,
  user_agent TEXT,
  metadata_json JSONB,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

## Core Flows

### 1) Login (without MFA)
1. Validate payload and rate-limit keys (`ip`, `email`).
2. Fetch user by email; if not found return generic unauthorized.
3. Verify hash using Argon2id.
4. Create `auth_sessions` row with new `refresh_jti`.
5. Return signed access token (15m) + refresh token (30d).
6. Emit `LOGIN_SUCCESS` event.

### 2) Refresh Token Rotation
1. Parse/verify refresh token signature and claims.
2. Lookup `refresh_jti` session and ensure not revoked/expired.
3. In single transaction: revoke old jti, create new jti row (or update with rotation chain).
4. Return new access+refresh tokens.
5. If reused old jti is detected, revoke entire user sessions (token theft defense).

### 3) Forgot/Reset Password
1. `forgot-password`: always return success; if account exists generate one-time reset token.
2. Store hashed reset token with TTL (e.g., 15 min) and send mail async.
3. `reset-password`: validate token hash + TTL + unused status.
4. Update password hash, mark token used, revoke all sessions.

## Concurrency, Consistency, and Idempotency

- Use unique index on `refresh_jti`; rotation done with transactional compare-and-swap semantics.
- `logout` idempotent: revoking already revoked token returns success.
- Email verification and reset tokens are one-time-use with atomic state transitions.

## Observability Checklist

- Metrics: login success rate, p95 latency, token refresh failure %, brute-force blocked count.
- Logs: structured auth events with correlation id (avoid PII in logs).
- Traces: gateway → auth service → db/cache/notification spans.
- Alerts: sudden spike in failures, unusual geo distribution, queue lag for verification/reset emails.
