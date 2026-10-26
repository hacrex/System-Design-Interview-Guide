# Case Study: URL Shortener

Try it yourself first using the [blank template](../templates/design-template.md). Then compare.

## 1. Requirements

**Functional**
- Create a short URL from a long URL
- Redirect a short URL to the original
- Optional: custom alias, expiry, click analytics

**Non-functional**
- Redirect latency under 100 ms at p99
- High availability for redirects (a dead link is a visible failure)
- Short codes must not be guessable in sequence if links are private
- Read heavy, roughly 100 reads per write

Out of scope for this round: user accounts, abuse review.

## 2. Estimation

From [module 06](../docs/06-estimation-and-metrics.md):
- 100M new URLs per month gives about 40 writes/s
- 100:1 read ratio gives about 4,000 reads/s average, 10,000/s peak
- About 500 bytes per record, so 50 GB per month and about 3 TB over 5 years

Conclusion: modest storage, read dominated. Cache heavily.

## 3. API

```
POST /v1/urls
  body: { "long_url": "...", "custom_alias": "optional", "expires_at": "optional" }
  returns: { "short_url": "https://sho.rt/aZ3x9Q" }

GET /{code}
  returns: 301 or 302 redirect to long_url
```

**301 vs 302:** 301 is permanent, so browsers cache it and your server sees fewer hits, but you lose analytics and cannot change the target. Use 302 if you need click counts or editable links.

## 4. Data model

Access pattern is a key lookup by code, so a key-value store fits well.

| Field | Notes |
|---|---|
| code (primary key) | 7 chars, base62 |
| long_url | Original URL |
| created_at, expires_at | For cleanup |
| owner_id | Optional |

7 base62 characters give 62^7, about 3.5 trillion combinations, far more than 100M per month needs.

## 5. High-level design

```
Client -> CDN -> Load balancer -> App servers -> Cache (Redis) -> Database
                                       |
                                  ID generator
                                       |
                                 Analytics queue -> Consumers -> Analytics store
```

**Read path:** check cache, on miss read the database, fill the cache, return the redirect.
**Write path:** generate a code, store the mapping, return the short URL.

## 6. Deep dive: generating the code

| Approach | How | Pros | Cons |
|---|---|---|---|
| Hash and truncate | MD5 or SHA of the URL, take 7 chars | Stateless | Collisions, needs retry on conflict |
| Random | Random base62 string | Not guessable | Collision check on every write |
| Counter and base62 | Global counter encoded in base62 | No collisions | Counter is a bottleneck, sequential and guessable |
| Pre-generated key service | Service hands out unused keys in ranges | Fast, no collisions | Extra component to run |

**Recommended:** Counter ranges. Each app server leases a block of IDs (say 10,000) from a coordinator, then encodes them locally in base62. To avoid guessable sequences, run the ID through a reversible shuffle before encoding. No collision checks, no per-request coordination.

## 7. Bottlenecks and tradeoffs

- **Cache hit rate:** Hot links dominate. LRU with a few GB handles most traffic.
- **Database:** Reads are mostly absorbed by cache. Shard by hash of code if storage grows. Replicate for availability.
- **Hot link:** A viral link can overload one cache node. Use local in-process caching for the top keys.
- **Expiry:** Lazy deletion on read plus a background cleanup job.
- **Abuse:** Rate limit creation per user and IP, scan for malicious targets.
- **Analytics:** Never block the redirect. Emit a click event to a queue and process it asynchronously.
- **Availability:** Multi-zone app and database, cache failure falls back to the database.

## Follow-up questions to practice

1. How do you support custom aliases without collisions?
2. How would you deliver redirects with low latency worldwide?
3. What changes if the read to write ratio becomes 1:1?
4. How do you count clicks accurately without slowing redirects?
