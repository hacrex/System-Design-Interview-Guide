# Short-notice cram plan

For when your interview is in **72 hours, 48 hours, or 24 hours**. The 8-week plan is ideal; this is triage. Be honest about what you can absorb and focus on the highest-signal items first.

## How to use

1. Pick the plan that matches your time left.
2. Follow the order — don't skip the framework (step 0). It's the skeleton every answer hangs on.
3. For each module, read it once, then answer the module's interview questions out loud.
4. Do one full timed case study (45 minutes) per day and score yourself with the rubric.
5. Stop and sleep. A tired brain misses failure modes and tradeoffs.

---

## 72-hour plan (3 days) — about 3–4 hours/day

### Day 1: Method + building blocks
- [ ] `docs/00-interview-framework.md` — memorize the 7 steps and time budget
- [ ] `docs/01-scaling-and-traffic.md` — scaling, load balancing, caching
- [ ] `docs/02-data-storage.md` — SQL vs NoSQL, sharding, consistent hashing
- [ ] Case study: [URL shortener](../case-studies/url-shortener.md) — timed 45 min, score with rubric

### Day 2: Distributed data + communication + reliability
- [ ] `docs/03-consistency-and-distributed-data.md` — CAP, consistency models, quorum, Saga
- [ ] `docs/04-communication.md` — queues, pub/sub, API design, gateway
- [ ] `docs/05-reliability-and-protection.md` — rate limiting, idempotency, retries, circuit breaker
- [ ] Case study: [rate limiter](../case-studies/rate-limiter.md) — timed 45 min, score with rubric

### Day 3: Estimation, observability, security + practice
- [ ] `docs/06-estimation-and-metrics.md` — latency numbers, QPS/storage recipe, p99
- [ ] `docs/07-observability-and-security.md` — RED/USE, authN/authZ, threats
- [ ] `docs/cheat-sheet.md` — skim the interview-signal table
- [ ] Case study: [key-value store](../case-studies/key-value-store.md) — timed 45 min, score with rubric
- [ ] `docs/grading-rubric.md` — review common mistakes

---

## 48-hour plan (2 days) — about 4–5 hours/day

### Day 1: Core framework + storage + communication
- [ ] `docs/00-interview-framework.md` — 7 steps, answer pattern, common mistakes
- [ ] `docs/01-scaling-and-traffic.md` — horizontal scaling, caching patterns, cache invalidation
- [ ] `docs/02-data-storage.md` — SQL vs NoSQL, shard key choice, consistent hashing
- [ ] `docs/04-communication.md` — queues vs direct HTTP, REST/gRPC/GraphQL, microservices vs monolith
- [ ] Case study: [URL shortener](../case-studies/url-shortener.md) — timed 45 min

### Day 2: Reliability + estimation + one more case study
- [ ] `docs/05-reliability-and-protection.md` — rate limiting, idempotency, circuit breaker, retries
- [ ] `docs/06-estimation-and-metrics.md` — numbers to memorize, estimation recipe, p99 vs average
- [ ] `docs/cheat-sheet.md` — interview-signal table for quick lookup
- [ ] Case study: [rate limiter](../case-studies/rate-limiter.md) OR [key-value store](../case-studies/key-value-store.md) — timed 45 min, score with rubric

---

## 24-hour plan (1 day) — about 6–8 hours, with breaks

### Morning: Framework + the three most-tested topics (3 hours)
- [ ] `docs/00-interview-framework.md` — 7 steps, time budget, answer pattern ("options → criteria → choice → tradeoff")
- [ ] `docs/01-scaling-and-traffic.md` — caching patterns + invalidation (read the table and failure modes)
- [ ] `docs/02-data-storage.md` — SQL vs NoSQL + sharding + consistent hashing
- [ ] `docs/03-consistency-and-distributed-data.md` — CAP + consistency models + Saga vs 2PC

### Midday: Reliability + communication (2 hours)
- [ ] `docs/05-reliability-and-protection.md` — rate limiting, idempotency, circuit breaker, retries with backoff
- [ ] `docs/04-communication.md` — queues vs HTTP, API design checklist, microservices vs monolith

### Afternoon: Estimation + observability + security + practice (2–3 hours)
- [ ] `docs/06-estimation-and-metrics.md` — memorize latency numbers, the 6-step estimation recipe
- [ ] `docs/07-observability-and-security.md` — RED/USE, authN vs authZ, top threats table
- [ ] `docs/cheat-sheet.md` — skim signal-to-module table
- [ ] One timed case study (45 min): pick [URL shortener](../case-studies/url-shortener.md) or [rate limiter](../case-studies/rate-limiter.md). Score with [rubric](../docs/grading-rubric.md).

### Evening: Final review (1 hour)
- [ ] Re-read your scored case study; drill the 0s and 1s in the matching module.
- [ ] Memorize the answer pattern: "I would use X because [reason]. The tradeoff is [cost]."
- [ ] Practice saying your scope-back sentence out loud: "I'm building X with Y constraints, intentionally out of scope: Z."
- [ ] Sleep. Rest beats cramming at 2 AM.

---

## If you only have 4 hours

| Minute | Topic |
|--------|-------|
| 0–30 | `docs/00-interview-framework.md` — the 7 steps |
| 30–60 | `docs/01-scaling-and-traffic.md` — caching + load balancing |
| 60–90 | `docs/02-data-storage.md` — SQL vs NoSQL + sharding |
| 90–120 | `docs/05-reliability-and-protection.md` — rate limiting + idempotency |
| 120–180 | Timed case study: [rate limiter](../case-studies/rate-limiter.md), then score |
| 180–240 | `docs/cheat-sheet.md` — signal table; review common mistakes |

## What to skip when time is short

- Deep proofs of Raft/Paxos (know the idea: majority quorum, terms, fencing tokens)
- Every edge case in the estimation examples (know the recipe, not every number)
- Building a full mock-interview bank (use [mock-interview-bank.md](../docs/mock-interview-bank.md) if you have a partner)

## Readiness check before the interview

- [ ] I can run the 7-step framework without notes
- [ ] I can estimate QPS and storage in 3 minutes
- [ ] I can compare SQL/NoSQL, REST/gRPC, 2PC/Saga without hesitating
- [ ] I name failure modes unprompted
- [ ] I state tradeoffs as "X because Y, at the cost of Z"
- [ ] I completed at least one full timed case study and scored it

Good luck.
