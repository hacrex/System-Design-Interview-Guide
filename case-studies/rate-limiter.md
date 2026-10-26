# Case Study: Rate Limiter

Try it yourself first using the [blank template](../templates/design-template.md). Then compare.

## 1. Requirements

**Functional**
- Limit requests per client (user, API key or IP) per time window
- Different limits per endpoint or plan
- Return a clear rejection when the limit is hit

**Non-functional**
- Very low overhead: adds only a few milliseconds per request
- Works across many app servers (distributed, shared counters)
- Highly available: the limiter must not take the API down
- Reasonably accurate, but small errors are acceptable

## 2. Estimation

- 1 million active clients, 100,000 requests/s at peak
- Per client state: a counter and a timestamp, about 50 to 100 bytes
- 1M clients x 100 bytes = about 100 MB of state, which fits in memory on one Redis node. Throughput, not size, is the concern.

## 3. Where to place it

| Placement | Pros | Cons |
|---|---|---|
| In each app server | No network hop | Limits are per server, not global |
| API gateway or middleware | One enforcement point, protects everything behind it | Gateway must scale |
| Dedicated service | Reusable by many teams | Extra network call |

**Recommended:** Enforce at the API gateway, backed by a shared Redis, with the rule configuration stored separately.

## 4. Algorithm choice

See the table in [module 05](../docs/05-reliability-and-protection.md). For this design:

**Token bucket** if bursts are fine, since it is simple and memory light (two values per client: tokens and last refill time).
**Sliding window counter** if you need smoother, stricter limits.

### Token bucket logic

```
on request(client):
    now = current_time()
    tokens, last = load(client)
    tokens = min(capacity, tokens + (now - last) * refill_rate)
    if tokens >= 1:
        tokens = tokens - 1
        save(client, tokens, now)
        allow
    else:
        reject with 429 and Retry-After
```

## 5. High-level design

```
Client -> Load balancer -> API gateway (limiter middleware)
                                 |            |
                           Redis cluster   Rules store
                                 |
                          Backend services
```

## 6. Deep dive: making it correct in a distributed setup

**Race condition:** Two servers read the same counter, both allow, both write. Fix with an atomic operation. Run the token bucket as a Redis Lua script so read, compute and write happen as one step.

**Redis capacity:** Shard keys by client ID with consistent hashing. A single client always maps to one node, so counters stay accurate.

**Multi-region:** Options are a global counter (accurate, slow), per-region limits (fast, a client can exceed the total by spreading across regions), or periodic syncing between regions. Most systems accept per-region limits with a slightly lower local cap.

**Failure behavior:** If Redis is unreachable, fail open (allow traffic, protect availability) for most APIs, or fail closed for sensitive ones such as login attempts. Make it configurable.

**Response:** Return `429 Too Many Requests` with `Retry-After`, plus headers such as `X-RateLimit-Limit` and `X-RateLimit-Remaining` so clients can back off properly.

## 7. Bottlenecks and tradeoffs

- **Hot client:** One heavy client hammers one Redis shard. Add a local pre-check or small in-memory bucket.
- **Clock skew:** Use the Redis server time, not app server clocks.
- **Rule changes:** Cache rules locally and refresh periodically.
- **Abuse via many IPs:** Combine rate limits with authentication and bot detection.
- **Observability:** Track allowed vs rejected counts per rule, and limiter latency.

## Follow-up questions to practice

1. How do you rate limit by both user and IP at once?
2. What happens at the window boundary with a fixed window counter?
3. How would you let premium customers have higher limits without redeploying?
4. Would you fail open or closed here, and why?
