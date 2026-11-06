# Self-grading rubric

Use this after you finish a case study with the [blank template](../templates/design-template.md). Score each of the seven framework steps from 0 to 2, then total it up. Be honest — the goal is to find weak spots before the real interview, not to feel good.

## Scoring

| Score | Meaning |
|-------|---------|
| 0 | Missing or wrong |
| 1 | Partial: mentioned, but shallow, vague, or missing a reason |
| 2 | Strong: clear, specific, with a tradeoff or reason stated |

**Total: /14.**  
- 12–14: interview-ready  
- 8–11: close, drill the 0s and 1s  
- Below 8: redo the case study before moving on

## 1. Clarify requirements (before you draw)

**Look for:**
- You named the top 3 functional features and explicitly said what is out of scope
- You stated non-functional targets: latency, availability, consistency, durability, read vs write ratio
- You said the scope back in one sentence and invited correction

**Signals:**
- "I would build X, Y, and Z. I am intentionally not doing accounts or analytics."
- "Redirects must be under 100 ms at p99. Reads are 100x writes."

## 2. Estimate scale (back-of-envelope)

**Look for:**
- You turned users into numbers: DAU, QPS (read and write), peak, storage per year, bandwidth, cache size
- You rounded aggressively (powers of 10) and said why the numbers matter (one box vs many)

**Signals:**
- "50,000 writes/s × 1 KB = 50 MB/s, about 4 TB/day. One machine cannot hold that, so we shard."
- You did not try to be exact; you used the estimate to drive a design decision.

## 3. Define the API (before the data model)

**Look for:**
- You listed endpoints/RPCs with inputs and outputs before choosing any database
- You mentioned pagination, authentication, or idempotency for writes where relevant
- You did not jump to boxes and arrows first

**Signals:**
- `POST /v1/urls { long_url } -> { short_url }`
- `GET /{code} -> 302 redirect`
- "I will make writes idempotent by requiring a client-generated request ID."

## 4. Model the data (driven by access patterns)

**Look for:**
- You named entities and the access patterns (point lookup, range scan, write-heavy, read-heavy)
- You picked SQL or NoSQL and gave a reason tied to the access pattern
- You did not design around tables you like; you designed around the reads and writes

**Signals:**
- "Access pattern is a key lookup by code, so a key-value store fits. SQL would add joins we do not need."
- "We need range scans on timestamp, so I will store by (user_id, created_at) rather than a flat key space."

## 5. High-level design (boxes and arrows, then read/write paths)

**Look for:**
- A simple diagram: client, load balancer, services, cache, database, queue as needed
- You walked one read request and one write request end to end
- You started simple, then scaled

**Signals:**
- "Client -> LB -> API server -> cache -> database. On a cache miss, read the database and fill the cache."
- You named where each piece lives (same zone? multi-zone? CDN edge?)

## 6. Deep dive (options, tradeoff, choice)

**Look for:**
- You picked 1–2 hard parts for this problem, not everything
- You showed at least two options with pros and cons
- You made a choice and stated what you give up (the tradeoff)

**Signals:**
- "Counter ranges give no collisions and no per-request coordination. The cost is an extra coordinator service to run."
- You did not say "X is the best" without explaining when it is wrong.

## 7. Bottlenecks and tradeoffs (failure, scale, cost)

**Look for:**
- Single points of failure
- Hot keys or hot partitions
- What happens when a dependency is slow or down
- Year 3 growth (data, QPS, regions)
- Monitoring and alerts you would add

**Signals:**
- "If Redis is unreachable, I fail open for reads and fail closed for writes. It is configurable per API."
- "A viral link can overload one cache node. I will add a small local in-process cache for the top keys."

## Follow-up handling (score separately, /4)

Interviewers often probe one area deeper. Give yourself 1 point for each you can answer off the cuff:

- Scale 10x: what breaks first, and what do you add?
- Scale 100x / multi-region: what changes in the data path?
- Region failover: how do you route traffic and keep data consistent?
- Cost: which component is the biggest bill, and how would you reduce it?

## Common mistakes (deduct mentally)

- Jumping to boxes before clarifying requirements
- Naming technologies without saying why
- Designing for Google scale when the numbers say otherwise
- Ignoring failure modes
- Talking for 10 minutes without checking in with the interviewer
- Picking the "best" tool without stating the tradeoff

## How to use

1. Do a case study with the blank template and a 45-minute timer.
2. Read your own answer and score each step 0/1/2.
3. For every 0 or 1, re-read the matching module (`docs/01-...` through `docs/07-...`) and redo that step.
4. Record yourself explaining one design. Count how many times you gave a reason — aim for a reason after every technology choice.
