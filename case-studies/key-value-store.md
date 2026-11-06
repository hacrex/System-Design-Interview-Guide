# Case Study: Key-value store

Try it yourself first using the [blank template](../templates/design-template.md). Then compare.

## 1. Requirements

**Functional**
- `put(key, value)` — store a value under a key
- `get(key)` — return the value for a key, or not found
- `delete(key)` — remove a key
- Keys and values are small (under 1 MB each), millions of keys
- Tunable consistency: a caller can choose strong or eventual consistency per request

**Non-functional**
- Reads and writes under 10 ms at p99 under normal load
- High availability: survive a node or zone failure without losing data
- Durability: acknowledged writes survive a node crash
- Linear scalability: add nodes to add capacity

Out of scope for this round: secondary indexes, range scans, transactions, multi-key operations.

## 2. Estimation

From [module 06](../docs/06-estimation-and-metrics.md):
- 10 million active clients, 50,000 writes/s and 500,000 reads/s at peak
- Average record (key + value + metadata) about 1 KB
- Writes: 50,000 x 1 KB = 50 MB/s, about 4 TB per day, roughly 1.5 PB per year
- Reads are 10x writes, so read traffic dominates

Conclusion: write throughput is the scaling bottleneck, not storage. Shard by key and replicate each shard for durability and read throughput.

## 3. API

```
PUT   /v1/kv/{key}
      body: { "value": "...", "consistency": "strong|eventual" }
      returns: 200 OK or error

GET   /v1/kv/{key}?consistency=strong|eventual
      returns: { "value": "..." } or 404

DELETE /v1/kv/{key}
      returns: 204 No Content or error
```

**Consistency parameter:** `strong` waits for a quorum of replicas to acknowledge before returning success. `eventual` returns as soon as the primary acknowledges, letting replication catch up asynchronously.

## 4. Data model

Access pattern is a point lookup by key, so partition by hash of the key.

| Field | Notes |
|---|---|
| key (partition key) | Hashed to pick a shard |
| value | Opaque blob, under 1 MB |
| version | Monotonically increasing, for conflict resolution |
| created_at, updated_at | For TTL and debugging |

Storage choice: a distributed key-value store built on top of per-node RocksDB or SQLite, replicated with a leader-follower protocol. Relational SQL is overkill (no joins, no transactions), and a document store adds overhead we don't need.

## 5. High-level design

```
Client -> Load balancer -> Coordinator / proxy
                             |
                +------------+------------+
                |                         |
            Shard A (leader)           Shard B (leader)
            - followers x2             - followers x2
                             |
                      Gossip / consensus for membership
```

Data is partitioned into shards by `hash(key) % num_shards`. Each shard has one leader and N followers. The coordinator routes `GET`/`PUT`/`DELETE` to the leader of the right shard.

**Read path:** client asks coordinator, coordinator forwards to shard leader. With `strong` consistency, read from leader (or a follower that has caught up to a known version). With `eventual`, read from any healthy follower for lower latency.

**Write path:** client sends `PUT` to coordinator, coordinator forwards to shard leader. Leader appends to its write-ahead log, replicates to followers, acknowledges once a quorum (leader + majority of followers) has persisted.

## 6. Deep dive: replication and consistency

This is the hard part. Two dimensions: how writes spread (replication), and how reads see them (consistency).

| Replication option | How it works | Pros | Cons |
|---|---|---|---|
| Single leader, async followers | Leader writes locally, streams changes to followers | Simple, fast writes | Followers can be stale; failover can lose un-replicated writes |
| Single leader, sync quorum | Leader waits for N-1 followers before ack | No acknowledged writes are lost | Slower writes, unavailable if quorum is down |
| Leaderless (Dynamo-style) | Any node can coordinate; writes go to N nodes, reads ask N nodes | No single leader bottleneck, highly available | Conflict resolution is needed; read repair and anti-entropy add complexity |

**Recommended:** Single leader per shard with sync quorum writes and tunable reads. A quorum of `W` out of `N` replicas must acknowledge a write; a read contacts `R` replicas and returns the newest version. If `W + R > N`, you get strong consistency (read your writes). Set `W = R = (N + 1) / 2` for a simple majority; lower `R` for faster, eventually-consistent reads.

**Conflict handling:** Each write carries a version (timestamp or logical clock). If two writes to the same key happen concurrently, the higher version wins (last write wins). For applications that cannot tolerate lost updates, expose the conflict to the caller or use application-level merge.

## 7. Bottlenecks and tradeoffs

- **Hot key:** One key hammered by many clients overloads its shard leader. Add a small local cache for read-heavy hot keys, or reject with retry for write-heavy hot keys.
- **Leader failure:** Use Raft or Paxos per shard so followers elect a new leader within seconds. Clients cache shard-to-leader mapping and refresh on failure.
- **Uneven load:** Some keyspace ranges are busier. Rebalance by splitting hot shards and moving them to less-loaded nodes.
- **Network partition:** With quorum writes, a minority partition cannot accept writes (sacrifices availability for consistency). Choose this because silent data divergence is worse than a temporary outage for a key-value store.
- **Year 3 growth:** 1.5 PB/year means adding nodes and re-sharding. Consistent hashing with virtual nodes makes rebalancing gradual instead of a big migration.
- **Monitoring and alerts:** Track per-shard write latency, replication lag (leader vs newest follower), quorum availability, and error rates by operation. Page on replication lag above threshold.

## Follow-up questions to practice

1. How do you handle range scans (get all keys with a prefix) in this design?
2. What changes if values can be 100 MB instead of 1 MB?
3. How would you add TTL (time-to-live) expiration without scanning every key?
4. How do you move a shard from one node to another without downtime?
5. Strong vs eventual consistency: when would a caller choose each?
