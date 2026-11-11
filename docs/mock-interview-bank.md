# Mock interview question bank

A consolidated list of timed prompts to simulate real rounds. Each question lists the module it exercises and a suggested difficulty. Run each with a 45-minute timer and score yourself with the [grading rubric](../docs/grading-rubric.md).

## How to use

1. Pick a question you haven't seen.
2. Set a 45-minute timer. Use the [blank template](../templates/design-template.md).
3. Work through the 7-step framework: clarify → estimate → API → data model → high-level design → deep dive → bottlenecks.
4. Score each step 0/1/2. Drill any 0s or 1s in the matching module.
5. Repeat with the follow-up variations ("scale 10x," "multi-region," "cost").

---

## Phase 1 — Method

| # | Prompt | Module | Focus | Difficulty |
|---|--------|--------|-------|------------|
| M1 | Design a URL shortener. | 00 | Full 7-step framework | Easy |
| M2 | Design a paste bin (like Pastebin). | 00 | Read-heavy, short URLs | Easy |
| M3 | Design a todo list API. | 00 | API design first, then boxes | Easy |

---

## Phase 2 — Building blocks (01–04)

| # | Prompt | Module | Focus | Difficulty |
|---|--------|--------|-------|------------|
| B1 | Design a rate limiter. | 01, 05 | Load balancing, caching, distributed counters | Easy |
| B2 | Design a CDN for image delivery. | 01 | Caching, edge, read path | Easy |
| B3 | Design a chat / messaging system storage. | 02 | SQL vs NoSQL, sharding, indexing | Medium |
| B4 | Design a key-value store. | 02 | Consistent hashing, partitioning | Medium |
| B5 | Design a distributed cache (like Redis/Memcached). | 02 | Consistent hashing, hot keys, eviction | Medium |
| B6 | Design a notification system. | 04 | Queues, pub/sub, retries, idempotency | Medium |
| B7 | Design a news feed. | 04 | Fan-out on write vs read, hot keys | Hard |
| B8 | Design an API gateway / BFF for mobile + web. | 04 | API design, gateway, auth | Medium |
| B9 | Design a chat system (WebSockets, presence, ordering). | 04 | Real-time, ordering, storage | Hard |
| B10 | Design a file storage and sync system (like Dropbox). | 02, 04 | Chunking, dedup, consistency, metadata | Hard |

---

## Phase 3 — Hard problems and applied skills (05–07)

| # | Prompt | Module | Focus | Difficulty |
|---|--------|--------|-------|------------|
| H1 | Design a payment system. | 05 | Idempotency, sagas, strong consistency | Hard |
| H2 | Design a retry policy for a payment call. | 05 | Retries, backoff, circuit breaker | Medium |
| H3 | Design a system for 99.99% availability. | 05 | SPOF, redundancy, failover | Medium |
| H4 | Design a metrics and monitoring system. | 07 | Time series storage, aggregation, alerting | Hard |
| H5 | Design authentication for a mobile app + public API. | 07 | OAuth 2.0, OIDC, JWT, sessions | Medium |
| H6 | Design a distributed transaction flow (checkout). | 03, 05 | 2PC vs Saga, outbox pattern | Hard |
| H7 | Design a leader election / coordination service. | 03 | Raft/Paxos, quorum, split brain | Hard |

---

## Phase 4 — Senior / staff follow-ups

These are harder and assume you already have a working design. Use them as follow-ups to any of the above.

| # | Prompt | What it tests |
|---|--------|---------------|
| S1 | Your design works in one region. How do you make it active-active across three regions? | Multi-region, data consistency, traffic routing |
| S2 | Traffic spikes 50x during a flash sale. What breaks first, and what do you add? | Scaling, bottlenecks, load shedding |
| S3 | A dependency you call is slow and causing timeouts cascading through your service. How do you stop it? | Circuit breaker, bulkheads, graceful degradation |
| S4 | Your p99 latency is 5 seconds but p50 is 100 ms. What do you check? | Tail latency, fan-out, percentiles |
| S5 | You need to cut infrastructure cost by 30%. Which component is the biggest bill, and how do you reduce it? | Cost optimization, storage tiers, reserved capacity |
| S6 | A node fails and a follower is promoted, but it was lagging — acknowledged writes are lost. How do you prevent this? | Replication, sync vs async, failover safety |
| S7 | How do you handle a "poison message" that crashes every consumer that reads it? | Dead letter queues, backpressure, observability |

---

## Timed practice plan

| Time | Activity |
|------|----------|
| 0–5 min | Clarify requirements (step 1). Say the scope back out loud. |
| 5–8 min | Estimate scale (step 2). DAU → QPS → storage → bandwidth → cache. |
| 8–13 min | Define the API (step 3). Endpoints, inputs, outputs, pagination, idempotency. |
| 13–18 min | Data model (step 4). Entities, access patterns, SQL vs NoSQL with reason. |
| 18–26 min | High-level design (step 5). Diagram + one read path + one write path. |
| 26–38 min | Deep dive (step 6). 1–2 hard parts, two options compared, choice + tradeoff. |
| 38–45 min | Bottlenecks and tradeoffs (step 7). SPOF, hot keys, failure behavior, year 3 growth, monitoring. |

After 45 minutes, stop — even if incomplete. That simulates the real interview. Then score yourself.
