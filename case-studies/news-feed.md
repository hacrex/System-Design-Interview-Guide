# Case Study: News Feed

Try it yourself first using the [blank template](../templates/design-template.md). Then compare.

## 1. Requirements

**Functional**
- A user can post an update (text, photo, link)
- A user sees a feed of recent posts from people they follow, ordered by recency
- A user can follow/unfollow other users
- Feed supports pagination (load more as you scroll)

**Non-functional**
- Feed must load within ~200 ms at p99
- Writes (posts) must be available and durable
- Reads are far more frequent than writes (roughly 100:1)
- Eventual consistency is acceptable (a post may take seconds to appear in followers' feeds)

Out of scope for this round: algorithmic ranking/relevance, ads, stories, groups, comments (keep it simple: posts only).

## 2. Estimation

- 100 million DAU
- Each user posts 1 update per day and follows 200 people on average
- Each user's feed is viewed 5 times per day

Writes:
- 100M posts/day → ~1,200 writes/s average, ~5,000/s peak

Reads:
- 100M users × 5 feed views/day = 500M feed reads/day → ~5,800 reads/s average, ~20,000/s peak
- Each feed read returns ~20 posts, so ~116M post fetches/day

Storage:
- Post: ~1 KB with metadata (text, media ref, author, timestamp)
- 100M × 1 KB = 100 GB/day, ~36 TB/year
- Follow graph: 100M × 200 follows = 20B edges; at ~50 bytes each ≈ 1 TB

Conclusion: read-heavy, fan-out is the core challenge. Precompute feeds (fan-out on write) so reads are cheap, and cache aggressively.

## 3. API

```
POST   /v1/posts
       body: { "author_id": "...", "content": "...", "media_url": "optional" }
       returns: { "post_id": "...", "created_at": "..." }

GET    /v1/feed?user_id=...&limit=20&cursor=...
       returns: { "posts": [...], "next_cursor": "..." }

POST   /v1/follows
       body: { "follower_id": "...", "followee_id": "..." }
       returns: 201 Created

DELETE /v1/follows/{followee_id}
       returns: 204 No Content
```

Pagination: cursor-based (by post ID or timestamp), not offset-based, because offset pagination is slow and inconsistent as new posts arrive.

## 4. Data model

Access patterns:
- Insert a new post (write-heavy)
- Fetch a user's followees (to fan out)
- Fetch a user's precomputed feed (read-heavy, latest first)

| Field | Notes |
|-------|-------|
| post_id (PK) | Snowflake ID or UUID |
| author_id | Indexed; used to fetch a user's posts |
| content, media_url | Post body and media reference |
| created_at | Sort key for feed ordering |
| followee_id / follower_id | Follow graph edges |

Storage choice:
- **Posts:** NoSQL wide-column or document store (key lookup by post ID, range scan by author + time). No joins needed.
- **Follow graph:** Graph-friendly store or sharded relational table keyed by follower_id.
- **Feed cache:** Redis sorted set per user (score = timestamp), holding the top ~1,000 precomputed posts per user.

## 5. High-level design

```
Client -> Load Balancer -> Post Service -> Post Storage
                              |
                          Follow Service -> Follow Storage
                              |
                          Feed Service -> Feed Cache (Redis)
                              |
                          Fan-out Workers -> Post Storage
```

Two writes happen when a user posts:
1. Post Service stores the post in Post Storage.
2. Feed Service fans the post out to each follower's feed cache.

**Write path (post):** Client → Post Service → Post Storage → Feed Service → fan out to followers' feed caches → return.

**Read path (feed):** Client → Feed Service → fetch from follower's feed cache (Redis sorted set) → return latest 20 posts.

---

## 6. Deep dive: fan-out on write vs fan-out on read

This is the hard part. When user A posts, how do followers see it?

| Approach | How it works | Pros | Cons |
|----------|--------------|------|------|
| Fan-out on write | On each post, push the post ID into every follower's feed cache. | Reads are O(1) cache lookups; fast feed load | Writes are expensive for users with millions of followers (celebrity problem) |
| Fan-out on read | At read time, fetch posts from all followees and merge-sort by time. | Writes are cheap; no celebrity problem | Reads are expensive; many followees to query, merge-sort adds latency |
| Hybrid | Fan-out on write for normal users; fan-out on read (or pull-based) for celebrities with huge follower counts. | Balances read and write cost | More complex; two code paths to maintain |

**Recommended:** Hybrid. For users with < 5,000 followers, fan-out on write (push to each follower's Redis sorted set). For celebrities (> 5,000 followers), do not push on write; instead, at read time, merge the celebrity's recent posts (pulled from Post Storage) into the follower's feed. This bounds write amplification while keeping reads fast.

**Cache structure:** Redis sorted set `feed:{user_id}` with score = post timestamp, members = post IDs. Keep the latest ~1,000 posts; trim older ones with `ZREMRANGEBYRANK`.

**Write amplification bound:** A user with 1M followers fan-out on write means 1M Redis writes per post. With hybrid, that user is treated as a celebrity and their posts are pulled at read time instead.

## 7. Bottlenecks and tradeoffs

- **Hot key / celebrity user:** A celebrity's post fans out to millions of feeds. Mitigation: hybrid approach (pull-based for celebrities) plus per-user write rate limits.
- **Feed cache miss:** If a user's feed cache is empty (cold start or eviction), fall back to fan-out on read by querying followees' recent posts and merging. Repopulate the cache asynchronously.
- **Follow/unfollow race:** When a user follows someone, backfill that person's recent posts into the follower's feed cache; when unfollowing, removal is lazy (filter at read time) to avoid expensive deletes.
- **Post storage growth:** 36 TB/year of posts; archive older posts to cheaper object storage and keep the last 30–90 days hot in the serving store.
- **Read amplification:** A feed read touches the cache once (O(1)), plus a database fetch for the full post content if not cached. Cache post content too for hot posts.
- **Year 3 growth:** Shard post storage and follow graph by `user_id`; shard feed caches by `user_id` as well. Add read replicas for Post Storage.
- **Monitoring and alerts:** Track feed cache hit rate, fan-out write latency (p99), follow graph read latency, and feed load p99/p999. Page on fan-out queue depth or cache miss rate spiking.

## Follow-up questions to practice

1. How do you support algorithmic ranking (relevance, not just recency) without killing read latency?
2. How would you handle a user with 10 million followers posting once — fan-out to all, or pull at read time?
3. What changes if you add photos/videos (large media) to posts?
4. How do you backfill a new user's feed when they follow someone with years of posts?
5. How do you handle feed reads during a Redis cache outage?
