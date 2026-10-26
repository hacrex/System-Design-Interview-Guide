# 07 Observability and Security

Concepts: observability (24), security basics (25).

## Observability

You cannot fix what you cannot see. Three pillars plus alerting:

| Pillar | Answers | Examples |
|---|---|---|
| Metrics | Is something wrong, and how much? | Request rate, error rate, latency percentiles, saturation |
| Logs | What happened? | Structured JSON logs with request IDs |
| Traces | Where did the time go across services? | Distributed tracing with a trace ID passed on every hop |

**Frameworks to cite:**
- **RED** for services: Rate, Errors, Duration
- **USE** for resources: Utilization, Saturation, Errors
- **Four golden signals:** latency, traffic, errors, saturation

**Good practice:**
- Alert on symptoms users feel (error rate, latency SLO burn), not every CPU spike
- Every alert needs a runbook and an owner
- Use structured logs and correlation IDs
- Keep metric label cardinality under control, or costs explode
- Health endpoints: liveness (restart me) vs readiness (send me traffic)

**In an interview:** After the design, add one minute on how you would know it is broken: dashboards, SLOs, and what pages someone at 3 AM.

## Security basics

**Authentication (who are you) vs authorization (what can you do)**

- **AuthN:** passwords with strong hashing (bcrypt, argon2), MFA, SSO.
- **OAuth 2.0:** Delegated access. A user grants an app limited access without sharing a password. Use the authorization code flow with PKCE for apps.
- **OpenID Connect:** Identity layer on top of OAuth 2.0.
- **Tokens:** JWTs are signed, self-contained, and hard to revoke early. Use short lifetimes plus refresh tokens. Sessions with server-side storage are easier to revoke.
- **AuthZ models:** RBAC (roles), ABAC (attributes). Enforce at the service, not only at the gateway.

**Protect data**
- **In transit:** TLS everywhere, including internal traffic (mTLS for service to service).
- **At rest:** Encrypt disks and databases, manage keys in a KMS, rotate them.
- **Secrets:** Never in code or images. Use a secrets manager.
- **PII:** Minimize, mask in logs, define retention.

**Common threats and one-line defenses**

| Threat | Defense |
|---|---|
| SQL injection | Parameterized queries |
| XSS | Output encoding, Content Security Policy |
| CSRF | SameSite cookies, CSRF tokens |
| Brute force and abuse | Rate limiting, lockout, bot detection |
| DDoS | CDN and edge protection, autoscaling, rate limits |
| Leaked credentials | Least privilege, short-lived credentials, rotation |

**Principles:** least privilege, defense in depth, secure defaults, audit logging.

## Interview questions

1. Your checkout latency doubled overnight. Walk through how you investigate.
2. How would you design authentication for a mobile app and a public API?
3. Where do you encrypt data, and who holds the keys?
4. What would you alert on for this system, and what would you leave out?
