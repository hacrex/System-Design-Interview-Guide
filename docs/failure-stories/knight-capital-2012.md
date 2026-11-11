# Failure Story: Knight Capital (August 2012)

## What happened

Knight Capital, a high-frequency trading firm, deployed new trading software to its production servers. A technician failed to deploy the new code to one of eight servers, and the old "Power Peg" code (which had been kept for a fallback) remained on that server. When the new code went live, the rogue server began sending millions of erroneous orders to the market.

Within 45 minutes, Knight Capital had sent 4 million trades across 150 stocks, accumulating roughly $440 million in losses. The firm nearly went bankrupt and was eventually acquired.

## Why it was so severe

- **Inconsistent deployment:** One server ran old code while the rest ran new code, creating a split-brain deployment.
- **No kill switch:** There was no fast way to stop the rogue server from sending orders once the problem was detected.
- **Old code left in place:** The deprecated "Power Peg" code was still loadable and executable, rather than being removed or disabled.
- **No canary:** The change was rolled out to all servers at once instead of being tested on a small subset first.

## The system-design lesson

**Deploys are a major source of incidents.** You need safe deployment practices and emergency controls.

Specifically:
- **Canary / blue-green deploys:** Roll changes to a small fraction of servers first, watch error rates, then proceed. See [module 05](../05-reliability-and-protection.md) on safe deploys.
- **Kill switch / circuit breaker:** Have a fast, one-click way to stop traffic to a faulty service or disable a feature.
- **Remove dead code:** Deprecated code paths should be deleted or disabled, not left in production where they can be reactivated by a config mistake.
- **Idempotency:** Where possible, make operations idempotent so retries don't compound errors. See [module 05 — Idempotency](../05-reliability-and-protection.md).
- **Observability on deploys:** Watch error rate and latency during and right after every deploy; pause on SLO burn.

## Interview angle

If asked "how would you deploy this change safely?":

> "I'd use a canary deploy: roll out to 5% of servers first, monitor error rate and latency, then increase to 100% if healthy. I'd also have a kill switch to disable the feature instantly if something goes wrong, and I'd make sure old code paths are removed rather than left in production."

## Related modules

- [Module 05 — Reliability and protection](../05-reliability-and-protection.md): safe deploys, circuit breakers, idempotency
- [Module 07 — Observability and security](../07-observability-and-security.md): alerts on SLO burn, dashboards
