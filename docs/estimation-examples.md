# Estimation worked examples

Companion to [module 06](06-estimation-and-metrics.md). Each example follows the six-step recipe: users → actions → QPS → storage → bandwidth → cache. Round aggressively; the goal is to size the system, not to be exact.

## Example 1: Photo sharing app

**Problem:** 50 million DAU, each user uploads 2 photos per day and views 20 photos per day. Estimate QPS and storage.

**Step 1 — Users:** 50M DAU.

**Step 2 — Actions per day:**
- Uploads: 50M × 2 = 100M writes/day
- Views: 50M × 20 = 1B reads/day

**Step 3 — QPS:**
- Write QPS: 100M / 100,000 seconds ≈ 1,000/s. Peak ≈ 3x average ≈ 3,000/s
- Read QPS: 1B / 100,000 seconds ≈ 10,000/s. Peak ≈ 30,000/s

**Step 4 — Storage:**
- Photo size: ~1 MB average (includes thumbnail metadata)
- Daily uploads: 100M × 1 MB = 100 TB/day — too large; compress to ~200 KB → 20 TB/day
- Per year: 20 TB × 365 ≈ 7 PB. With 3x replication: ~21 PB over one year

**Step 5 — Bandwidth:**
- Read bandwidth: 10,000 reads/s × 200 KB = 2 GB/s sustained
- Upload bandwidth: 1,000 writes/s × 200 KB = 200 MB/s

**Step 6 — Cache:** Hot 20% of daily read data: 20 TB × 20% = 4 TB/day. A few TB of Redis/CDN covers the hot set; the rest goes to object storage.

**Conclusion to state out loud:** Reads dominate by 10x and storage is in the petabyte range, so the design priority is object storage plus a CDN for hot photos, not a relational database.

---

## Example 2: Chat / messaging app

**Problem:** 20M DAU, each user sends 30 messages per day and reads 200 messages per day. Average message is 500 bytes.

**Step 1 — Users:** 20M DAU.

**Step 2 — Actions per day:**
- Sends: 20M × 30 = 600M writes/day
- Reads: 20M × 200 = 4B reads/day

**Step 3 — QPS:**
- Write QPS: 600M / 100,000 ≈ 6,000/s. Peak ≈ 15,000/s
- Read QPS: 4B / 100,000 ≈ 40,000/s. Peak ≈ 100,000/s

**Step 4 — Storage:**
- Daily messages: 600M × 500 bytes = 300 GB/day
- Per year: 300 GB × 365 ≈ 110 TB. With 3x replication: ~330 TB/year

**Step 5 — Bandwidth:**
- Read bandwidth: 40,000 × 500 bytes = 20 MB/s sustained — small
- Write bandwidth: 6,000 × 500 bytes = 3 MB/s

**Step 6 — Cache:** Recent messages are hot. Cache the last 50 messages per active chat; a few hundred GB of Redis covers most reads.

**Conclusion to state out loud:** Read QPS is very high (100k/s peak) but each read is tiny, so the bottleneck is connection handling and fan-out to many devices, not storage. Shard by user_id and cache aggressively.

---

## Example 3: Why p99, not average?

**Setup:** 1,000 requests. Latencies (ms): 999 requests at 50 ms, 1 request at 5,000 ms.

- Average: (999 × 50 + 5,000) / 1,000 ≈ 55 ms. Sounds fine.
- p50: 50 ms.
- p99: 5,000 ms. One in a hundred users waits 5 seconds.

**Takeaway:** The average hides the tail. p99 tells you what the worst-case experience is for nearly all users, and tail latency is what users actually notice. In fan-out systems (call 5 services), the slowest call dominates, so p99 compounds quickly.

---

## Example 4: Throughput is fine, users complain about slowness

**What to check:**

1. **Look at percentiles, not average throughput.** Throughput can be high while p99 latency is terrible (see Example 3). A few slow requests do not lower overall throughput much but ruin the user experience.
2. **Check for tail-latency causes:** GC pauses, lock contention, disk jitter, cross-region round trips, or a slow dependency called synchronously.
3. **Check fan-out:** One request calling multiple services sequentially adds latencies. If any dependency is slow, the whole request is slow.
4. **Check queueing:** If thread pools or connection pools are saturated, requests wait in line. Little's Law: at 1,000 req/s with 200 ms latency, ~200 requests are in flight. If your pool is smaller than that, requests queue and latency spikes.
5. **Check the slow path:** Errors and retries often hide in the 99th percentile. A service returning 500s that clients retry turns one slow call into three.

**What to do:** Add p95/p99 dashboards and alerts, trace slow requests end to end, cache hot data, move work off the critical path (async), and right-size connection/thread pools.

---

## How to use

When you hit an estimation question in an interview:

1. State your assumptions out loud (users, actions per day, object size).
2. Walk the six steps, rounding to powers of 10.
3. Say what the numbers mean for the design: one box or many? Cache or not? SQL or object storage?
4. End with a one-sentence conclusion, just like the examples above.
