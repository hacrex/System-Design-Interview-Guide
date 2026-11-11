# Common interview mistakes (quick-reference)

A one-page list of the most frequent system-design interview anti-patterns, why each hurts your signal, and what to do instead. Each entry cross-references the module where the topic is covered.

Use this as a pre-interview checklist: read it, then score yourself on a practice case study with the [grading rubric](grading-rubric.md).

## The mistakes

| # | Mistake | Why it hurts | What to do instead | Module |
|---|---------|--------------|--------------------|--------|
| 1 | **Jumping to boxes before clarifying requirements** | You design the wrong system; the interviewer can't correct your scope. | Ask functional/non-functional questions first, then say the scope back in one sentence. | [00 Framework](00-interview-framework.md) |
| 2 | **Naming technologies without saying why** | Sounds like you're reciting a stack, not reasoning. | Every tech choice gets a reason tied to access pattern, scale, or failure mode. | [All modules] |
| 3 | **Designing for Google scale when the numbers say otherwise** | Over-engineering; you miss that one box might be enough. | Run the estimation first, then let the numbers drive sharding, caching, and storage choices. | [06 Estimation](06-estimation-and-metrics.md) |
| 4 | **Ignoring failure modes** | Your design falls apart when a dependency slows or dies. | Name SPOFs, hot keys, and what happens when a dependency is down; add circuit breakers and graceful degradation. | [05 Reliability](05-reliability-and-protection.md) |
| 5 | **Talking for 10 minutes without checking in** | You may go down a wrong path unnoticed; loses the interviewer. | Pause every few minutes: "Does this direction sound right?" or "Shall I dive deeper here?" | [00 Framework](00-interview-framework.md) |
| 6 | **Picking the "best" tool without stating the tradeoff** | Shows you don't understand that every choice has a cost. | Show options, pros/cons, your pick, and what you give up: "X because Y, at the cost of Z." | [All modules] |
| 7 | **Forgetting idempotency on writes** | Retries cause double charges, duplicate emails, or corrupted state. | Require an idempotency key on write APIs and dedupe at the storage layer. | [05 Idempotency](05-reliability-and-protection.md) |
| 8 | **Overlooking cache invalidation** | Users see stale data; you hand-wave "just use Redis." | Name an invalidation strategy: short TTL, explicit delete on write, or versioned keys. | [01 Caching](01-scaling-and-traffic.md) |
| 9 | **Choosing NoSQL because it's "modern"** | Wrong tool for the access pattern; you lose joins or transactions you need. | Start relational unless key-based access at scale clearly demands NoSQL. | [02 Data storage](02-data-storage.md) |
| 10 | **Using offset pagination for large or changing data** | Offset gets slow and inconsistent as data grows. | Prefer cursor-based pagination keyed by timestamp or ID. | [04 API design](04-communication.md) |
| 11 | **No rate-limit / fail-open vs fail-closed decision** | Your limiter becomes a new SPOF or takes the API down. | Decide per API: fail open (availability) or fail closed (protection); return 429 + Retry-After. | [05 Rate limiting](05-reliability-and-protection.md) |
| 12 | **Quoting average latency instead of p99** | Average hides the tail that ruins user experience. | Use p50/p95/p99; mention tail latency compounds in fan-out calls. | [06 Metrics](06-estimation-and-metrics.md) |
| 13 | **Synchronous cross-service calls on the critical path** | One slow dependency cascades and takes down your service. | Use async queues, circuit breakers, bulkheads, and graceful degradation. | [04 Queues](04-communication.md), [05 Reliability](05-reliability-and-protection.md) |
| 14 | **Alerting on CPU instead of user symptoms** | You get paged for false positives and miss real user impact. | Alert on error rate and SLO burn (RED/golden signals), not every metric spike. | [07 Observability](07-observability-and-security.md) |
| 15 | **Storing secrets in code or images** | Credentials leak; security failure before the design even starts. | Use a secrets manager; never commit keys, tokens, or internal hostnames. | [07 Security](07-observability-and-security.md) |

## The answer pattern for any tradeoff

Use this for every technology choice:

> "I would use **[option]** because **[reason tied to this workload]**. The tradeoff is **[what you give up]**, which is acceptable because **[why it's fine here]**."

Example:
> "I would use **eventual consistency** for the feed because reads are 100× writes and users tolerate a few seconds of staleness. The tradeoff is that a user may not see their own post immediately, which is acceptable for a feed."

## How to use

1. **Before a mock interview:** skim this list once.
2. **After a case study:** score yourself with the [grading rubric](grading-rubric.md), then check which mistakes you made.
3. **During the real interview:** if you catch yourself doing one of these, correct it out loud — interviewers value self-correction.
