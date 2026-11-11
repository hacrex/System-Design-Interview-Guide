# 05 Reliability and Protection

Concepts: rate limiting (13), idempotency (17), availability (23), plus failure handling.

## Contents

- [Availability](#availability)
- [Rate limiting](#rate-limiting)
- [Idempotency](#idempotency)
- [Failure handling](#failure-handling)
- [Interview questions](#interview-questions)

## Availability

Availability is the fraction of time the system serves requests correctly.

| Target | Downtime per year | Per month |
|---|---|---|
| 99% | 3.65 days | 7.3 hours |
| 99.9% | 8.76 hours | 43.8 minutes |
| 99.99% | 52.6 minutes | 4.4 minutes |
| 99.999% | 5.3 minutes | 26 seconds |

**Terms:** SLI is what you measure, SLO is the target, SLA is the contractual promise with penalties. An **error budget** is the allowed unreliability that you can spend on releases.

**Serial dependencies multiply:** three services at 99.9% each give about 99.7% end to end. Parallel redundancy raises availability: two independent 99% copies give 99.99%.

**How to raise availability:**
- Remove single points of failure (load balancer, database, DNS, a single zone)
- Redundancy across zones, then regions if needed
- Health checks and automatic failover
- Graceful degradation: serve cached or partial results instead of an error
- Safe deploys: canary, blue green, quick rollback

**Disaster recovery:** RPO is how much data you can lose, RTO is how long you can be down. Strategies from cheap to costly: backup and restore, pilot light, warm standby, active-active.

## Rate limiting

Protects a system from abuse and overload and makes usage fair.

| Algorithm | Idea | Notes |
|---|---|---|
| Fixed window | Count per time window | Simple, but allows a burst at window edges |
| Sliding window log | Store timestamps | Accurate, memory heavy |
| Sliding window counter | Weighted blend of two windows | Good balance |
| Token bucket | Tokens refill at a rate, each request spends one | Allows bursts, most common |
| Leaky bucket | Fixed outflow rate | Smooths traffic |

**Where:** API gateway, per service, or both. **Key by:** user, API key, IP, or endpoint.

**Distributed concerns:** Keep counters in Redis with atomic operations (Lua script or `INCR` with expiry). Decide fail-open vs fail-closed when the limiter is down. Return `429` with `Retry-After`.

See the [rate limiter case study](../case-studies/rate-limiter.md).

## Idempotency

An operation is idempotent if repeating it has the same effect as doing it once. Essential because networks retry.

**Naturally idempotent:** `PUT`, `DELETE`, setting a value.
**Not idempotent:** `POST /charge`, incrementing a counter.

**Idempotency key pattern:**
1. Client sends a unique key with the request.
2. Server stores the key and result in the same transaction as the work.
3. A repeat with the same key returns the stored result instead of redoing the work.

Keep keys for a bounded window and reject a reused key with a different payload. At-least-once queues plus idempotent consumers give effectively exactly-once behavior.

## Failure handling

- **Timeouts:** Always set them. A missing timeout turns a slow dependency into an outage.
- **Retries with exponential backoff and jitter:** Prevents retry storms. Cap attempts. Retry only idempotent or safe operations.
- **Circuit breaker:** After repeated failures, stop calling the dependency for a cooldown, then probe. States: closed, open, half-open.
- **Bulkheads:** Isolate resource pools so one failing dependency cannot exhaust everything.
- **Load shedding and backpressure:** Reject low priority work early when overloaded rather than failing everything.
- **Graceful degradation:** Fallbacks such as cached data or a simplified feature.

## Interview questions

1. Your SLA is 99.95%. What does your architecture need to look like?
2. Design a retry policy for a payment call. What can go wrong?
3. A client retries a charge after a timeout. How do you prevent a double charge?
4. Compare token bucket and sliding window counter.
