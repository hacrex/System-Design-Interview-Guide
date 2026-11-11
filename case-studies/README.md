# Practice Problems

Work through these with the [blank template](../templates/design-template.md). Solved examples so far: [URL shortener](url-shortener.md), [rate limiter](rate-limiter.md), [key-value store](key-value-store.md), [notification system](notification-system.md), [news feed](news-feed.md), [payment system](payment-system.md), [video streaming](video-streaming.md), [chat system](chat-system.md), [distributed cache](distributed-cache.md), [file storage and sync](file-storage-sync.md), [search autocomplete](search-autocomplete.md), [web crawler](web-crawler.md), [ride hailing](ride-hailing.md), and [metrics and monitoring system](metrics-monitoring.md).

## Suggested order

| # | Problem | Main concepts it exercises | Status |
|---|---------|---------------------------|--------|
| 1 | URL shortener | ID generation, caching, read heavy design | Solved |
| 2 | Rate limiter | Algorithms, atomicity, distributed counters | Solved |
| 3 | Key-value store | Replication, quorums, consistent hashing | Solved |
| 4 | Notification system | Queues, pub/sub, retries, idempotency | Solved |
| 5 | News feed | Fan-out on write vs read, caching, hot keys | Solved |
| 6 | Chat system | Websockets, ordering, storage, presence | Solved |
| 7 | Distributed cache | Eviction, consistent hashing, hot keys | Solved |
| 8 | File storage and sync | Chunking, dedup, consistency, metadata | Solved |
| 9 | Video streaming | CDN, encoding pipeline, storage tiers | Solved |
| 10 | Payment system | Idempotency, sagas, strong consistency, reconciliation | Solved |
| 11 | Search autocomplete | Tries, ranking, caching, update pipeline | Solved |
| 12 | Web crawler | Queues, politeness, dedup, partitioning | Solved |
| 13 | Ride hailing | Geo indexing, matching, real-time updates | Solved |
| 14 | Metrics and monitoring system | Time series storage, aggregation, alerting | Solved |

## How to add a case study

1. Copy `templates/design-template.md`.
2. Fill in all seven framework sections.
3. Include numbers in the estimation, one deep dive with options compared, and a list of tradeoffs.
4. Add follow-up questions at the end.
5. Link it from this table and from the main README.

## Practice tip

Score yourself before you compare with the solved examples: see [docs/grading-rubric.md](../docs/grading-rubric.md). It maps to the same seven framework steps in the template, so you can spot your 0s and 1s and drill them before moving on.
