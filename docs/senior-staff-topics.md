# Senior / staff-level interview topics

Beyond the 25 core concepts, senior and staff interviews probe operational depth, cross-region design, cost, and incident judgment. These topics are not a separate module — they are follow-ups you should be able to layer onto any case study.

## 1. Multi-region and active-active design

**What changes when you go from one region to three?**

- **Traffic routing:** Use a global load balancer (DNS, anycast, or a managed GSLB) to route users to the nearest region. Plan for failover: if a region goes dark, route its traffic to healthy regions.
- **Data replication:** Async replication between regions is the norm (latency makes sync impractical). Accept eventual consistency cross-region; use strong consistency only within a region.
- **Write conflicts:** The same data written in two regions can diverge. Resolve with last-write-wins, vector clocks, or application-level merge. Expose conflicts to the caller when automatic resolution is unsafe.
- **Data residency:** Some data (PII, payments) may be legally required to stay in a specific region. Tag and route accordingly.
- **Failure blast radius:** A region should fail independently. Avoid cross-region synchronous calls on the critical path; use async queues instead.

**Interview angle:** Say "I'd keep the control plane regional and replicate data asynchronously between regions, accepting eventual consistency cross-region because synchronous cross-region writes would add hundreds of milliseconds of latency."

---

## 2. Capacity planning and cost optimization

**Which component is the biggest bill, and how do you reduce it?**

- **Storage tiers:** Hot data (last 30–90 days) in fast storage; warm data in cheaper object storage; cold data in archive (Glacier, etc.). Define retention and move data automatically.
- **Reserved vs spot vs on-demand:** Steady baseline workloads → reserved instances; bursty, fault-tolerant batch work → spot; unpredictable traffic → on-demand with autoscaling.
- **Right-sizing:** Don't over-provision for peak if you can use autoscaling + queues to smooth load. Monitor utilization (CPU, memory, connections) and shrink oversized instances.
- **Data transfer:** Cross-region and cross-AZ traffic costs money. Keep traffic within a region/AZ where possible; use caching and CDN at the edge.
- **Index and query cost:** Unused indexes slow writes and cost storage. Review query patterns and drop indexes that aren't used.

**Interview angle:** End your design with a one-minute cost note: "The biggest bill will be object storage for photos. I'd move older photos to a cold storage tier and use a CDN so users don't hit origin for every view."

---

## 3. Blast radius and cellular architecture

**How do you prevent one failure from taking down everything?**

- **Cells / shards as isolation units:** Split customers or traffic into independent cells. A failure in one cell doesn't affect others. Route by customer ID or region to a cell.
- **Bulkheads:** Isolate resource pools (connection pools, thread pools, message queues) per dependency so one slow dependency can't exhaust everything.
- **Load shedding:** Reject low-priority work early when overloaded rather than failing everything. Return 503 or 429 for non-critical traffic.
- **Graceful degradation:** Serve cached or partial results instead of an error. Example: show the last-known feed instead of failing to load it.
- **Canary and blue-green deploys:** Roll changes to a small percentage first; watch error budgets before proceeding.

**Interview angle:** Name the blast radius explicitly: "I'll shard by user_id so a hot user or a bad deploy can only affect one cell, not the whole system."

---

## 4. Observability for distributed systems

**How do you know it's broken, and where?**

- **Distributed tracing:** Pass a trace ID on every hop. In a fan-out call (one request → five services), tracing tells you which service is slow.
- **Correlation IDs:** Include a request ID in every log line so you can follow a single request across services.
- **SLI/SLO/error budget:** Measure what users feel (error rate, latency SLO burn). Spend the error budget on releases; pause deploys when the budget is exhausted.
- **Alert on symptoms, not metrics:** Page on error rate and latency SLO, not on every CPU spike. Every alert needs a runbook and an owner.
- **Health endpoints:** Liveness ("restart me") vs readiness ("send me traffic"). Use both for safe rolling deploys.

**Interview angle:** After your design, add one minute: "I'd add RED metrics (rate, errors, duration) per service, distributed tracing with a trace ID, and an alert on p99 latency above 500 ms with a runbook."

---

## 5. Incident judgment and postmortems

**What do you do when things go wrong?**

- **Stop the bleeding first:** Roll back, fail over, or shed load before investigating root cause.
- **Blameless postmortem:** Focus on what happened and how to prevent recurrence, not who made a mistake.
- **Timeline:** Build a minute-by-minute timeline of detection, response, and recovery.
- **Action items:** Concrete, owned, dated follow-ups (not vague "we should be more careful").
- **Communication:** Status page updates, customer emails, internal stakeholder briefings — with clear owners and cadence.

**Interview angle:** If asked about a past outage, describe: symptom → detection → mitigation → root cause → prevention. Be specific and own your part.

---

## How to layer these onto a case study

After you finish the 7-step framework, pick one follow-up:

- "Scale 10x: what breaks first?" → apply **capacity planning** and **blast radius**
- "Multi-region: what changes?" → apply **multi-region design**
- "How do you know it's broken?" → apply **observability**
- "A dependency is slow and cascading." → apply **circuit breaker / bulkheads / load shedding**

These follow-ups are what separate a senior answer from a mid-level one.
