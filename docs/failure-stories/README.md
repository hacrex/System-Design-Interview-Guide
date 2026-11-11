# Real-World Failure Stories

Short write-ups of actual outages and incidents: what failed, the design lesson, and how to apply it in an interview. These are useful for the "bottlenecks and tradeoffs" step and senior-level follow-ups.

## Story list

| # | Outage | Lesson | Module |
|---|--------|--------|--------|
| 1 | [AWS S3 us-east-1 (2017)](s3-2017.md) | Don't depend on a single service or region without circuit breakers and graceful degradation | 01, 05 |
| 2 | [Knight Capital (2012)](knight-capital-2012.md) | Canary deploys, kill switches, and idempotency prevent costly rollouts | 05 |
| 3 | [Cloudflare global outage (2022)](cloudflare-2022.md) | A bad deploy can take down everything — blast radius, canary, and rollback matter | 05, 07 |

## How to use in an interview

- Mention one of these when discussing failure modes in step 7 (bottlenecks and tradeoffs).
- When asked "how do you make this reliable?" cite a real failure and the specific control that would have prevented or limited it.
- When asked about deploys, reference canary/blue-green and kill switches from Knight Capital or Cloudflare.

## Contributing a failure story

See [CONTRIBUTING.md](../CONTRIBUTING.md). A good story:

- names a real, publicly reported incident,
- explains what failed in 3–5 sentences,
- states the system-design lesson,
- links to a public postmortem if available.
