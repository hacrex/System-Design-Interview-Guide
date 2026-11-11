# Failure Story: Cloudflare Global Outage (June 2022)

## What happened

Cloudflare deployed a change to its WAF (Web Application Firewall) rule engine. The change included a poorly written regular expression that caused excessive CPU usage — catastrophic backtracking — when it matched certain traffic patterns. The rule was deployed globally to Cloudflare's edge network.

Within seconds, CPU usage spiked across Cloudflare's edge locations worldwide, and the network began dropping legitimate traffic. Large portions of the internet became unreachable through Cloudflare. The outage lasted roughly 47 minutes before Cloudflare could roll back the change.

## Why it was so severe

- **Global deploy:** The bad change went to all edge locations at once, so there was no healthy region to fall back to.
- **No canary:** The rule wasn't tested on a small subset of traffic or locations first.
- **Blast radius:** A single regex in one WAF rule affected the entire global edge network.
- **Rollback took time:** Identifying the offending rule and reverting it globally took nearly an hour.

## The system-design lesson

**Blast radius matters.** A change to one component should not be able to take down the entire system.

Specifically:
- **Canary deploys:** Roll out rule or code changes to a small percentage of traffic or a single location first. See [module 05](../05-reliability-and-protection.md) on safe deploys.
- **Circuit breakers on dependencies:** Edge services should trip a breaker if a downstream rule engine or dependency becomes slow, failing fast rather than consuming all CPU. See [module 05 — circuit breaker](../05-reliability-and-protection.md).
- **Resource isolation (bulkheads):** Limit CPU/memory per tenant, per rule, or per location so one bad rule can't starve everything else.
- **Fast rollback:** Keep the ability to revert a change globally in seconds, not minutes.
- **Observability:** Alert on CPU saturation and error rate spikes per location so you detect the problem immediately. See [module 07](../07-observability-and-security.md).

## Interview angle

If asked "how do you prevent a bad deploy from taking down everything?":

> "I'd use canary deploys — push to one location or 1% of traffic first, watch CPU and error rates, then expand. I'd add circuit breakers so a slow dependency doesn't consume all resources, and I'd isolate workloads with bulkheads so one bad rule can't starve the whole edge network. I'd also have a one-click global rollback."

## Public postmortem

Cloudflare published "Understanding the June 2, 2022 Cloudflare Outage" on their blog. Search for it to read the technical details.

## Related modules

- [Module 05 — Reliability and protection](../05-reliability-and-protection.md): canary deploys, circuit breakers, bulkheads
- [Module 07 — Observability and security](../07-observability-and-security.md): alerts, dashboards, RED metrics
