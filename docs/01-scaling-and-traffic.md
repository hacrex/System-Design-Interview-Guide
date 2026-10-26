# 01 Scaling and Traffic

Concepts: scalability (1), load balancing (2), caching (3).

## Scalability

| | Vertical (scale up) | Horizontal (scale out) |
|---|---|---|
| What | Bigger machine | More machines |
| Pros | Simple, no code change | No hard ceiling, fault tolerant |
| Cons | Hard limit, single point of failure, downtime to resize | Needs stateless services, data partitioning, more ops |
| Use when | Early stage, databases that are hard to shard | Web and app tiers, anything growing fast |

**Interview angle:** Make the app tier stateless first (sessions in Redis or signed tokens), then scale it horizontally. Scale the data tier last because it is the hard part.

## Load balancing

Spreads requests across healthy servers and removes unhealthy ones.

- **L4 (transport):** Routes on IP and port. Fast, protocol agnostic.
- **L7 (application):** Routes on path, header, cookie. Enables canary releases and per-route policies.
- **Algorithms:** Round robin, least connections, weighted, IP hash, consistent hash (good for caches).
- **Health checks:** Active (probe) and passive (watch failures). Tune thresholds or you will flap.
- **Avoid a new single point of failure:** Run load balancers in pairs or use a managed one across zones.

## Caching

Serve hot data from fast memory to cut latency and protect the database.

**Where to cache:** browser, CDN, API gateway, application (Redis, Memcached), database buffer pool.

| Pattern | How it works | Good for | Risk |
|---|---|---|---|
| Cache-aside | App checks cache, on miss reads DB and fills cache | General reads | Stale data, miss stampede |
| Read-through | Cache loads from DB itself | Simpler app code | Cache is a dependency |
| Write-through | Write to cache and DB together | Read-after-write needs | Slower writes |
| Write-back | Write to cache, flush to DB later | Write heavy, loss tolerant | Data loss on crash |

**Eviction:** LRU is the default answer. LFU for stable popularity. TTL always.

**Invalidation is the hard part.** Options: short TTLs, explicit delete on write, versioned keys.

**Failure modes to mention:**
- **Stampede:** Many requests miss at once. Fix with request coalescing or a lock, plus jittered TTLs.
- **Penetration:** Requests for keys that do not exist. Fix with negative caching or a bloom filter.
- **Hot key:** One key overloads one node. Fix with local replicas or key splitting.

## Interview questions

1. Your service hits 10x traffic. What do you change first, and why?
2. When would sticky sessions be acceptable, and what do they cost you?
3. How do you keep a cache consistent with the database after a write?
4. Explain a cache stampede and two ways to prevent it.
