# Case Study: Notification System

Try it yourself first using the [blank template](../templates/design-template.md). Then compare.

## 1. Requirements

**Functional**
- Send a notification to a user (email, push, SMS, in-app)
- Support multiple channels per notification
- Track delivery status (queued, sent, delivered, failed, read)
- Retry failed deliveries with backoff
- Idempotent: resending the same notification does not duplicate it

**Non-functional**
- Notifications must reach the user within seconds under normal load
- High availability: survive a channel provider outage without losing notifications
- Exactly-once delivery semantics per notification attempt
- Handle traffic spikes (flash sales, breaking news)

Out of scope for this round: notification preferences UI, unsubscribe management, A/B testing of message content.

## 2. Estimation

- 50 million DAU, each user receives 5 notifications per day on average
- 250M notifications/day → ~2,900 writes/s average, ~10,000/s peak (3–4x)
- Average payload: ~200 bytes (title, body, deep link, metadata)
- Daily storage: 250M × 200 bytes ≈ 50 GB/day; ~18 TB per year
- Reads: users fetch their notification list, dominated by recent items

Conclusion: write throughput and fan-out to multiple channels are the bottlenecks, not storage. Use a queue to absorb spikes and workers per channel.

## 3. API

```
POST   /v1/notifications
       body: {
         "user_id": "...",
         "channels": ["push", "email"],
         "title": "...",
         "body": "...",
         "deep_link": "optional",
         "idempotency_key": "...",
         "schedule_at": "optional ISO timestamp"
       }
       returns: { "notification_id": "...", "status": "queued" }

GET    /v1/notifications/{id}
       returns: { "id": "...", "status": "delivered", "channel": "push", ... }

GET    /v1/users/{user_id}/notifications?limit=20&cursor=...
       returns: list of notifications with pagination cursor
```

Idempotency key: client-generated UUID; the server rejects reuse with a different payload.

## 4. Data model

Access patterns:
- Insert a new notification (write-heavy)
- Fetch a user's recent notifications (read-heavy, latest first)
- Update delivery status as providers acknowledge

| Field | Notes |
|-------|-------|
| notification_id (PK) | UUID or Snowflake ID |
| user_id | Indexed; used to fetch a user's feed |
| channels | Array / set of target channels |
| title, body, deep_link | Message content |
| idempotency_key | Unique per logical send; indexed |
| status | queued, sending, sent, delivered, failed, read |
| created_at, updated_at | For TTL, sorting, debugging |
| provider_message_id | Returned by channel provider (SendGrid, FCM, Twilio) |

Storage choice: PostgreSQL for the notification record (needs status transitions and idempotency checks in a transaction), plus a message queue (Kafka or RabbitMQ) for fan-out to channel workers.

## 5. High-level design

```
Client -> API Gateway -> Notification Service -> Postgres (notification records)
                                              |
                                          Queue (Kafka / RabbitMQ)
                                              |
                    +-------------------------+-------------------------+
                    |                         |                         |
             Push Worker              Email Worker              SMS Worker
                    |                         |                         |
              FCM / APNs              SendGrid / SES            Twilio
                                              |
                                       Delivery status callback
                                              |
                                       Postgres (update status)
```

**Write path:** client calls API → service validates idempotency key → inserts record as `queued` → publishes to queue → returns `notification_id`.

**Read path:** fetch recent notifications for a user from Postgres, ordered by `created_at`, cursor-paginated.

**Status updates:** channel providers call a webhook or the worker polls; service updates the record to `sent`/`delivered`/`failed`.

## 6. Deep dive: exactly-once delivery and retries

This is the hard part. Networks retry, providers retry, and workers crash. We need each notification to be delivered once per channel.

| Approach | How it works | Pros | Cons |
|----------|--------------|------|------|
| Idempotency key + transaction | Client sends a key; service checks it inside the same DB transaction that inserts the notification. Repeat with the same key returns the stored result. | Safe retries, no duplicate sends | Requires client to generate and store keys |
| Outbox pattern | Service writes notification + outbox event in one transaction; a relay publishes the event to the queue. | Decouples DB from queue; no dual-write failure | Extra table and relay process to run |
| At-least-once queue + idempotent worker | Queue guarantees delivery ≥1; worker checks `idempotency_key` before calling provider and marks the notification as already processed. | Simple, robust to worker crashes | Duplicate provider calls possible if worker crashes after send but before ack |

**Recommended:** Idempotency key + transaction at the API boundary, plus at-least-once queue with an idempotent worker. The worker records the `provider_message_id` after a successful send; a retry with the same key sees the record and skips the provider call.

**Retry policy:** exponential backoff with jitter, capped at 5–6 attempts. Move permanently failed notifications to a dead-letter queue (DLQ) for manual inspection. Do not retry non-idempotent operations (e.g., charging) — but notifications are safe to retry if the provider supports idempotency keys too.

## 7. Bottlenecks and tradeoffs

- **Hot user:** A celebrity with millions of followers triggers a fan-out storm. Cap fan-out per request; push bulk sends to a background job with lower priority.
- **Channel provider outage:** If FCM is down, queue push notifications and retry later; degrade gracefully by sending email/SMS instead where configured.
- **Rate limits per channel:** Providers throttle by API key. Track per-provider quota and back off; use multiple sender IDs / IPs for SMS.
- **Ordering:** A user may see a "read" notification before a newer "sent" one if workers race. Sequence by `created_at` on read, not by delivery completion.
- **Year 3 growth:** 18 TB/year of records; archive older notifications to cheaper storage (S3/Glacier) and keep only the last 90 days hot in Postgres.
- **Monitoring and alerts:** Track enqueue vs. send vs. deliver rates per channel, provider latency, DLQ depth, and idempotency-key collision rate. Page on delivery rate dropping below SLO.

## Follow-up questions to practice

1. How do you schedule a notification for a future time without polling?
2. How would you handle a user who wants quiet hours (no push between 11 PM and 7 AM)?
3. What changes if you add in-app notifications with real-time WebSocket delivery?
4. How do you deduplicate when the same event (e.g., "post liked") triggers notifications to many users?
