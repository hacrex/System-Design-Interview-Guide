# 06 Estimation and Performance Metrics

Concepts: back-of-envelope estimation (21), latency vs throughput (22).

## Numbers worth memorizing

**Time**
- 1 day is about 86,400 seconds, call it 100,000 (10^5) for quick math
- 1 month is about 2.5 million seconds
- 1 year is about 30 million seconds

**Latency (rough orders of magnitude)**

| Operation | Time |
|---|---|
| Memory read | ~100 ns |
| SSD random read | ~100 microseconds |
| Same datacenter round trip | ~0.5 ms |
| Disk seek (HDD) | ~10 ms |
| Cross-continent round trip | ~100 to 150 ms |

**Sizes**
- 1 KB = 10^3 bytes, 1 MB = 10^6, 1 GB = 10^9, 1 TB = 10^12, 1 PB = 10^15
- Tweet or short text row: ~1 KB with metadata
- Photo: ~200 KB to 2 MB. Short video minute: tens of MB.

## Estimation recipe

1. **Users:** Total users, daily active users (DAU).
2. **Actions:** Actions per user per day, split by read and write.
3. **QPS:** `DAU x actions per day / 86,400`. Peak is roughly 2x to 5x average.
4. **Storage:** `writes per day x size x 365 x years x replication factor`.
5. **Bandwidth:** `QPS x average payload size`.
6. **Cache:** Cache the hot 20% of daily read data as a first guess.

### Worked example: URL shortener

- 100 million new URLs per month, read:write = 100:1
- Write QPS: 100M / 2.5M seconds = about 40/s
- Read QPS: 40 x 100 = about 4,000/s, peak maybe 10,000/s
- Storage: 100M x 500 bytes = 50 GB per month, about 3 TB over 5 years
- Cache: if 20% of 4,000 reads/s are unique hot data, a few GB of Redis covers most hits

**Conclusion to state out loud:** Reads dominate, data fits comfortably on a small cluster, so the design priority is low-latency reads and cache, not exotic storage.

## Latency vs throughput

- **Latency:** Time for one request. Measure percentiles (p50, p95, p99), not the average. Tail latency drives user experience, and fan-out calls amplify it.
- **Throughput:** Requests handled per second.
- They trade off: batching and queues raise throughput but add latency. Parallelism raises throughput but may add contention.

**Little's Law:** `concurrent requests = throughput x latency`. If you handle 1,000 req/s at 200 ms, about 200 requests are in flight, which sizes thread pools and connection pools.

**Improving latency:** cache, CDN, move compute near the user, avoid sequential calls, use async work, add indexes.
**Improving throughput:** scale out, batch, use queues, reduce per-request work, connection pooling.

## Interview questions

1. Estimate storage and QPS for a photo sharing app with 50 million DAU.
2. Why is p99 more useful than the average?
3. Your throughput is fine but users complain about slowness. What do you check?

Worked answers to all three: [estimation-examples.md](estimation-examples.md).
