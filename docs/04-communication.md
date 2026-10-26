# 04 Communication

Concepts: message queues (10), pub/sub (11), API design (12), API gateway (16), microservices vs monolith (15).

## Message queues

A queue decouples producer and consumer in time and speed.

**Why use one:** absorb traffic spikes, retry failed work, scale consumers independently, smooth load on a slow dependency.

- **Delivery guarantees:** at-most-once (may lose), at-least-once (may duplicate, the common default), exactly-once (really at-least-once plus idempotent processing).
- **Ordering:** Usually guaranteed only within a partition or key. Choose the partition key to keep related events together.
- **Dead letter queue (DLQ):** Park messages that keep failing so they do not block the rest.
- **Backpressure:** When consumers fall behind, queue depth grows. Monitor lag and scale consumers, or shed load.

**Kafka vs a traditional queue (SQS, RabbitMQ):**

| | Kafka (log) | Queue (SQS, RabbitMQ) |
|---|---|---|
| Model | Durable replayable log, consumer groups track offsets | Messages removed once acknowledged |
| Strength | High throughput, replay, many consumers | Simple task distribution, per-message routing |
| Use for | Event streams, analytics, event sourcing | Background jobs, work queues |

## Pub/sub

One event, many independent subscribers. The publisher does not know who listens.

- Topics fan out to many consumer groups. Each group gets every message, and consumers inside a group share the load.
- Good for notifications, search index updates, cache invalidation, audit logs.
- Watch for: schema evolution (version your events), poison messages, and subscribers that silently fall behind.

## API design

| | REST | gRPC | GraphQL |
|---|---|---|---|
| Transport | HTTP and JSON | HTTP/2 and Protobuf | HTTP, single endpoint |
| Strength | Simple, cacheable, universal | Fast, typed contracts, streaming | Client picks fields, fewer round trips |
| Weakness | Over and under fetching | Poor browser support, harder to debug | Caching is harder, risk of expensive queries |
| Best for | Public APIs | Internal service to service | Varied clients such as mobile and web |

**Design checklist:**
- Pagination: prefer cursor based over offset for large or changing data
- Versioning strategy decided up front
- Idempotency keys for writes that must not repeat
- Consistent error format and status codes
- Rate limits and authentication documented

## API gateway

A single front door for clients.

- **Does:** authentication, routing, rate limiting, TLS termination, request logging, response caching, protocol translation.
- **Does not do:** business logic. Keep it thin.
- **Risk:** It becomes a bottleneck and a single point of failure, so run it redundantly.
- **BFF (backend for frontend):** A gateway tailored per client type, such as mobile or web.

## Microservices vs monolith

| | Monolith | Microservices |
|---|---|---|
| Deploy | One unit | Independent per service |
| Team scaling | Hard past a point | Teams own services |
| Complexity | In the code | In the network and operations |
| Data | One database, easy transactions | Database per service, sagas needed |
| Failure | All or nothing | Partial, but cascades possible |

**Honest advice:** Start with a well-structured (modular) monolith. Split when team size, deploy cadence or scaling needs justify the operational cost. In an interview, explain why you are splitting, not just that you are.

**Service-to-service concerns:** service discovery, timeouts, retries with backoff, circuit breakers, tracing across hops.

## Interview questions

1. When would you pick a queue over a direct HTTP call between services?
2. How do you handle a consumer that keeps failing on one message?
3. REST, gRPC or GraphQL for a mobile app backed by ten services?
4. What would make you break up a monolith, and what would make you not?
