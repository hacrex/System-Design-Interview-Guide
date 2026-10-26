# System Design Template

Copy this file and fill it in for each practice problem. Set a 45 minute timer.

**Problem:**
**Date:**
**Time taken:**

## 1. Requirements (5 min)

Functional (top 3):
-
-
-

Non-functional (latency, availability, consistency, durability):
-
-

Out of scope:
-

## 2. Estimation (3 min)

- Users / DAU:
- Read vs write ratio:
- Write QPS / Read QPS / Peak:
- Storage per year:
- Bandwidth:
- Cache size:

What does this tell me about the design?

## 3. API (5 min)

```
```

## 4. Data model (5 min)

Entities:
Access patterns:
Storage choice and why:

## 5. High-level design (8 min)

Diagram (ASCII or sketch):

```
```

Read path:
Write path:

## 6. Deep dive (12 min)

Hard part 1:
- Option A:
- Option B:
- Choice and tradeoff:

Hard part 2:

## 7. Bottlenecks and tradeoffs (7 min)

- Single points of failure:
- Hot keys or partitions:
- Dependency failure behavior:
- Year 3 growth:
- Monitoring and alerts:

## Self review

- Did I clarify requirements before drawing?
- Did every technology choice have a reason?
- Did I cover failure modes?
- What would I do differently?
