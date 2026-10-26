# 02 Data Storage

Concepts: SQL vs NoSQL (6), indexing (9), replication (5), sharding (4), partitioning (18), consistent hashing (14).

## SQL vs NoSQL

| | SQL (relational) | NoSQL |
|---|---|---|
| Model | Tables, joins, schema | Key-value, document, wide-column, graph |
| Strength | Transactions, flexible queries, integrity | Horizontal scale, flexible schema, high write throughput |
| Weakness | Harder to scale writes | Limited joins, query patterns fixed up front |
| Examples | PostgreSQL, MySQL | DynamoDB, Cassandra, MongoDB, Redis |

**Rule of thumb:** Start relational unless the access pattern or scale clearly demands otherwise. Pick NoSQL for known key-based access at large scale, not because it sounds modern.

## Indexing

An index speeds reads by keeping a sorted structure (usually a B-tree) beside the table.

- **Cost:** Extra storage and slower writes, since every write updates each index.
- **Composite indexes:** Column order matters. Put equality columns first, range columns last.
- **Covering index:** Contains every column the query needs, so the table is never touched.
- **LSM trees** (Cassandra, RocksDB): Write optimized, reads may check several levels.
- **Do not index:** Low cardinality columns, or tables with heavy writes and few reads.

## Replication

Copies of the same data on multiple nodes for read scale and durability.

- **Leader-follower:** Writes go to the leader, reads can go to followers. Simple, but followers can lag.
- **Multi-leader:** Writes in several regions. Needs conflict resolution.
- **Leaderless (quorum):** Any node accepts writes. Reads and writes use quorums (R + W > N for strong reads).
- **Sync vs async:** Sync is safer but slower and less available. Async is fast but can lose recent writes on failover.

See [03](03-consistency-and-distributed-data.md) for lag and failover.

## Sharding vs partitioning

The terms overlap in practice. A clean way to separate them:

- **Partitioning:** Split a large table into pieces, often within one database. Keeps indexes small and lets you drop old data cheaply (for example by month).
- **Sharding:** Split data across multiple machines so each holds a subset. This scales writes and storage.

**Shard key choice is the whole game.**

| Strategy | Pros | Cons |
|---|---|---|
| Range | Range queries are easy | Hot spots (for example latest timestamps) |
| Hash | Even spread | Range queries hit every shard |
| Directory (lookup) | Flexible | Lookup service is a dependency |

**Problems to name:** cross-shard joins and transactions, resharding pain, hot shards, celebrity users. Prefer a key that matches the main access pattern.

## Consistent hashing

Place nodes and keys on a ring. A key belongs to the next node clockwise. When a node is added or removed, only the keys next to it move, instead of nearly everything with `hash(key) % N`.

- **Virtual nodes:** Each physical node owns many ring positions, which evens out load and lets stronger machines take more.
- **Used in:** Distributed caches, Dynamo-style stores, load balancing for cache affinity.

## Hot keys and hot partitions

One key or shard gets disproportionate traffic. Mitigations: add a random suffix and aggregate on read, cache the hot item, split the shard, or rate limit per key.

## Interview questions

1. Design the storage for a chat app. SQL or NoSQL, and what is the shard key?
2. Why does `hash(key) % N` hurt when you add a server, and how does consistent hashing fix it?
3. You added an index and writes got slower. Why?
4. How would you reshard a live database without downtime?
