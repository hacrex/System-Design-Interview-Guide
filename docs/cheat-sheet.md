# Cheat sheet & glossary

A one-page reference for when you are mid-preparation and need to find the right module fast.

## If you hear this in the interview, read this module

| Interview signal | Module | Why |
|---|---|---|
| "Traffic is spiking, one server can't keep up" | [01 Scaling and traffic](01-scaling-and-traffic.md) | Horizontal vs vertical scaling, load balancers, caching |
| "Data is too big for one box" or "we need to route by key" | [02 Data storage](02-data-storage.md) | Sharding, partitioning, consistent hashing, SQL vs NoSQL |
| "Reads are slow and repetitive" | [01 Scaling and traffic](01-scaling-and-traffic.md) | Caching strategies and cache invalidation |
| "The system must stay consistent during a failure" | [03 Consistency and distributed data](03-consistency-and-distributed-data.md) | CAP theorem, consistency models, replication lag, failover |
| "Two services must agree on an outcome" | [03 Consistency and distributed data](03-consistency-and-distributed-data.md) | Distributed transactions (2PC vs Saga), quorum, leader election |
| "Services need to talk to each other" | [04 Communication](04-communication.md) | Queues, pub/sub, API design, API gateway, microservices vs monolith |
| "Clients are hammering one endpoint" | [05 Reliability and protection](05-reliability-and-protection.md) | Rate limiting, idempotency, retries with backoff, circuit breakers |
| "We need to know how big this is" | [06 Estimation and metrics](06-estimation-and-metrics.md) | Back-of-envelope QPS, storage, bandwidth, latency vs throughput |
| "How do we know it's broken?" or "secure the endpoints" | [07 Observability and security](07-observability-and-security.md) | Monitoring, alerting, security basics |

## Glossary: extra topics beyond the 25 concepts

These come up in senior interviews. They are covered inside modules 03 and 05 but listed here for quick lookup.

**Quorum** — A write or read is only considered successful once a majority (W out of N replicas, or R out of N) has acknowledged it. If W + R > N, you get strong consistency; if W + R ≤ N, you trade consistency for availability. See module 03.

**Leader election** — When the leader of a shard or service dies, replicas run an election (often Raft or Paxos) to pick a new one. Clients must be able to discover the new leader. See module 03.

**Hot key** — One key or partition receives far more traffic than others (a viral link, a celebrity's feed). It overloads a single node. Fixes: local caching for the top keys, shard splitting, or rejecting write-heavy hot keys. See modules 01, 03, 05.

**Backpressure** — When a downstream service is slow, it signals upstream to slow down or reject new work instead of queueing forever and running out of memory. Implemented with full queues, windowing, or HTTP 429/503. See module 05.

**Retries with backoff** — A client retries a failed request, but waits progressively longer between attempts (exponential backoff, often with jitter) so it does not turn a small outage into a cascading failure. See module 05.

**Circuit breaker** — After N consecutive failures to a dependency, the client "trips the breaker" and fails fast for a cooldown period instead of retrying, protecting both itself and the struggling dependency. After the cooldown, it tries again half-open. See module 05.

## The 25 concepts at a glance

| # | Concept | Module |
|---|---------|--------|
| 1 | Scalability (horizontal vs vertical) | 01 |
| 2 | Load balancing | 01 |
| 3 | Caching | 01 |
| 4 | Sharding | 02 |
| 5 | Replication | 02 |
| 6 | SQL vs NoSQL | 02 |
| 7 | CAP theorem | 03 |
| 8 | Consistency models | 03 |
| 9 | Indexing | 02 |
| 10 | Message queues | 04 |
| 11 | Pub/sub | 04 |
| 12 | API design (REST, gRPC, GraphQL) | 04 |
| 13 | Rate limiting | 05 |
| 14 | Consistent hashing | 02 |
| 15 | Microservices vs monolith | 04 |
| 16 | API gateway | 04 |
| 17 | Idempotency | 05 |
| 18 | Partitioning | 02 |
| 19 | Replication lag and failover | 03 |
| 20 | Distributed transactions (2PC vs Saga) | 03 |
| 21 | Back-of-envelope estimation | 06 |
| 22 | Latency vs throughput | 06 |
| 23 | Availability | 05 |
| 24 | Observability | 07 |
| 25 | Security basics | 07 |

## How to use

- Stuck on a problem? Find the interview signal above and open the matching module.
- Heard a term you don't recognize? Check the glossary.
- Practicing a case study? Keep this tab open next to the [blank template](../templates/design-template.md) and the [grading rubric](grading-rubric.md).
