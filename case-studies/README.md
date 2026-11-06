# Practice Problems

Work through these with the [blank template](../templates/design-template.md). Solved examples so far: [URL shortener](url-shortener.md), [rate limiter](rate-limiter.md), and [key-value store](key-value-store.md).

## Suggested order

| # | Problem | Main concepts it exercises | Status |
|---|---------|---------------------------|--------|
| 1 | URL shortener | ID generation, caching, read heavy design | Solved |
| 2 | Rate limiter | Algorithms, atomicity, distributed counters | Solved |
| 3 | Key-value store | Replication, quorums, consistent hashing | Solved |
| 4 | Notification system | Queues, pub/sub, retries, idempotency | Practice |
| 5 | News feed | Fan-out on write vs read, caching, hot keys | Practice |
| 6 | Chat system | Websockets, ordering, storage, presence | Practice |
| 7 | Distributed cache | Eviction, consistent hashing, hot keys | Practice |
| 8 | File storage and sync | Chunking, dedup, consistency, metadata | Practice |
| 9 | Video streaming | CDN, encoding pipeline, storage tiers | Practice |
| 10 | Payment system | Idempotency, sagas, strong consistency, reconciliation | Practice |
| 11 | Search autocomplete | Tries, ranking, caching, update pipeline | Practice |
| 12 | Web crawler | Queues, politeness, dedup, partitioning | Practice |
| 13 | Ride hailing | Geo indexing, matching, real-time updates | Practice |
| 14 | Metrics and monitoring system | Time series storage, aggregation, alerting | Practice |

## How to add a case study

1. Copy `templates/design-template.md`.
2. Fill in all seven framework sections.
3. Include numbers in the estimation, one deep dive with options compared, and a list of tradeoffs.
4. Add follow-up questions at the end.
5. Link it from this table and from the main README.
