# Case Study: Chat System

Try it yourself first using the [blank template](../templates/design-template.md). Then compare.

## 1. Requirements

**Functional**
- Users can send and receive one-on-one and group messages
- Real-time message delivery (websocket)
- Message history: view past messages in a conversation
- Presence: show online/offline status and last seen
- Message delivery status: sent, delivered, read

**Non-functional**
- Low latency: message delivery under ~200 ms at p99
- Message ordering per conversation must be preserved
- High availability: chat must keep working during a single zone failure
- Eventually consistent for delivery receipts is acceptable

Out of scope for this round: end-to-end encryption, voice/video calls, reactions, stickers, message editing/deletion, read receipts in groups at scale.

## 2. Estimation

- 50 million DAU, each user sends 20 messages per day and receives 30
- 1B messages/day → ~12,000 writes/s average, ~50,000/s peak
- Average message: ~200 bytes (text, sender, timestamp, conversation ID)
- Storage: 1B × 200 bytes = 200 GB/day; ~73 TB/year
- Concurrent websocket connections: assume 5% of DAU online at once = 2.5M concurrent connections

Conclusion: connection management and message fan-out (especially group chat) are the bottlenecks, not storage. Use websockets for real-time delivery and a message queue for offline/async delivery.

## 3. API

```
POST   /v1/conversations
       body: { "participant_ids": ["...", "..."], "is_group": false, "name": "optional" }
       returns: { "conversation_id": "...", "created_at": "..." }

POST   /v1/conversations/{conversation_id}/messages
       body: { "sender_id": "...", "content": "...", "media_url": "optional", "reply_to": "optional" }
       returns: { "message_id": "...", "conversation_id": "...", "created_at": "...", "sequence_number": "..." }

GET    /v1/conversations/{conversation_id}/messages?limit=20&cursor=...
       returns: list of messages with cursor

WS     /v1/connect?user_id=...&token=...
       bidirectional websocket for real-time messages and presence

GET    /v1/users/{user_id}/presence
       returns: { "user_id": "...", "status": "online|offline|away", "last_seen": "..." }
```

## 4. Data model

Access patterns:
- Insert a new message (write-heavy, ordered per conversation)
- Fetch recent messages for a conversation (read-heavy, cursor-paginated)
- Fetch a user's conversations (read-heavy)
- Update presence (frequent writes)

| Entity | Field | Notes |
|--------|-------|-------|
| Conversation | conversation_id (PK) | Unique conversation identifier |
| Conversation | is_group, name, created_at | Group flag, display name |
| ConversationParticipant | conversation_id, user_id, joined_at, last_read_at | Many-to-many membership |
| Message | message_id (PK) | Snowflake ID or UUID |
| Message | conversation_id | Indexed; used to fetch conversation history |
| Message | sender_id | Indexed |
| Message | content, media_url, reply_to | Message body |
| Message | sequence_number | Monotonic per conversation for ordering |
| Message | created_at | Sort key |
| UserPresence | user_id (PK) | status, last_seen, connection_id |

Storage choice: relational database sharded by `conversation_id` (one conversation lives on one shard, so ordering is easy). Presence in Redis (fast reads/writes, TTL for stale connections).

## 5. High-level design

```
Client (websocket) -> Load Balancer -> Connection Manager (websocket nodes)
                                                      |
                                              Message Service (stateless)
                                                      |
                                        +-------------+-------------+
                                        |                           |
                                Conversation DB (shard by conv_id)  Presence (Redis)
                                        |
                                Message Queue (Kafka/RabbitMQ)
                                        |
                              Offline Delivery Workers -> Push/Email
```

**Write path (real-time):** sender's websocket → connection manager → message service → persist to conversation DB → assign sequence_number → publish to message queue → connection manager pushes to recipients' websocket connections (if online) → queue handles offline recipients → return ack to sender.

**Read path (history):** client → message service → conversation DB (cursor-paginated by created_at/sequence_number) → return messages.

**Presence:** websocket connect/disconnect updates Redis with TTL; subscribers (friends) get presence updates via pub/sub.

## 6. Deep dive: message ordering and delivery guarantees

This is the hard part. In a distributed chat system, how do we keep messages ordered per conversation while scaling horizontally?

| Approach | How it works | Pros | Cons |
|----------|--------------|------|------|
| Sequence number per conversation | DB shard assigns a monotonic sequence_number per conversation on insert. | Simple, strict ordering per conversation | Single shard becomes a bottleneck for hot conversations |
| Log per conversation (Kafka partition) | Each conversation maps to a Kafka partition; consumers read in offset order. | Strict ordering, scalable write throughput | Rebalancing is painful; partition count must be planned |
| Vector clocks / causal ordering | Each message carries a clock; clients merge by causal history. | No central coordinator, eventually consistent | Complex; clients must reconcile out-of-order delivery |

**Recommended:** Shard conversation DB by `conversation_id`. Since all messages for one conversation go to the same shard, the DB can assign a monotonic sequence_number, giving strict per-conversation ordering. For hot group conversations, add a per-conference message queue partition so writes are serialized by that partition.

**Delivery guarantees:** At-least-once (message queue with acks) is the default. Idempotent message processing (dedup by message_id) makes retries safe. Exactly-once is not worth the cost for chat — a rare duplicate message is acceptable.

**Websocket connection management:** 2.5M concurrent connections cannot all live on one node. Use a connection manager cluster with a pub/sub layer (Redis or Kafka) so any node can deliver a message to any online user. Route by user_id to a specific connection node using consistent hashing, avoiding sticky sessions.

## 7. Bottlenecks and tradeoffs

- **Hot conversation:** A viral group chat with thousands of active members fans out every message to many connections. Mitigation: cap group size, fan out asynchronously, use presence to skip offline members, backpressure if a member's connection is slow.
- **Connection storage:** 2.5M websocket connections consume memory. Use connection pooling, idle timeouts, and distribute across connection manager nodes.
- **Offline message delivery:** Users who were offline miss websocket pushes. The message queue delivers to push notification / email workers asynchronously; messages wait in the queue until delivered.
- **Message ordering across reconnects:** If a websocket drops, the client reconnects and may miss messages. Client sends a `last_received_sequence_number` on reconnect; server replays missed messages from the DB.
- **Presence scale:** Frequent presence updates (typing indicators, online/offline) can overwhelm Redis. Batch presence updates and use pub/sub channels per user's friend list.
- **Year 3 growth:** 73 TB/year of messages; shard conversation DB further, archive old conversations to cold storage, keep only recent history hot.
- **Monitoring and alerts:** Track message delivery latency (p99/p999), websocket connection count, queue depth for offline delivery, failed delivery rate, and presence update rate. Page on delivery latency spikes or queue backlog.

## Follow-up questions to practice

1. How do you support typing indicators without overwhelming the server?
2. What changes if you add end-to-end encryption (server cannot read message content)?
3. How do you handle message edit/delete with real-time delivery?
4. How do you scale presence for users with tens of thousands of friends (celebrities)?
5. How do you prevent a malicious user from opening millions of websocket connections?
