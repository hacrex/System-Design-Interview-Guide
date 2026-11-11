# 03 Consistency and Distributed Data

Concepts: CAP (7), consistency models (8), replication lag and failover (19), distributed transactions (20).

## Contents

- [CAP theorem](#cap-theorem)
- [Consistency models](#consistency-models)
- [Quorums](#quorums)
- [Leader election and consensus](#leader-election-and-consensus)
- [Replication lag and failover](#replication-lag-and-failover)
- [Distributed transactions](#distributed-transactions)
- [Interview questions](#interview-questions)

## CAP theorem

When a network partition happens, a system must choose:

- **Consistency (C):** Every read sees the latest write, or an error.
- **Availability (A):** Every request gets a response, possibly stale.

Partition tolerance is not optional in a distributed system, so the real choice is C or A during a partition. When there is no partition you can have both.

- **CP examples:** Systems that refuse requests rather than return stale data (ZooKeeper, etcd, many relational setups with synchronous replication). Good for money, inventory, locks.
- **AP examples:** Systems that keep answering and reconcile later (Cassandra, DynamoDB defaults). Good for feeds, carts, counters.

**PACELC** extends this: even without partitions, you trade latency against consistency.

## Consistency models

From strongest to weakest:

| Model | Meaning | Typical use |
|---|---|---|
| Linearizable | Behaves like a single copy, real-time order | Locks, leader election, balances |
| Sequential | Everyone sees the same order, not tied to real time | Some coordination services |
| Causal | Cause appears before effect | Comments and replies |
| Read-your-writes | You always see your own updates | Profile edits |
| Eventual | Replicas converge given no new writes | Likes, view counts, feeds |

**Interview angle:** Say where each part of the system needs which level. Payments strong, feed eventual. Mixed answers show maturity.

## Quorums

With N replicas, write to W and read from R. If R + W > N, reads overlap the latest write. Common setting: N=3, W=2, R=2. Lower R or W for speed, accept staleness.

## Leader election and consensus

Raft and Paxos let a group agree on a leader and an ordered log despite failures. Know the idea, not the proofs:

- A majority (quorum) must agree, so a cluster of 2f+1 nodes survives f failures.
- Terms or epochs prevent two leaders acting at once (split brain).
- Fencing tokens stop an old leader from writing after it lost leadership.

Used by etcd, Consul, ZooKeeper (ZAB), and many databases.

## Replication lag and failover

**Lag:** Followers trail the leader. Symptoms: a user posts, refreshes, and the post is missing.

Fixes:
- Read-your-writes by routing a user's reads to the leader for a short window
- Monotonic reads by pinning a user to one replica
- Use synchronous or semi-synchronous replication for critical data

**Failover steps:** detect leader failure, elect a new leader, repoint clients, reconcile the old leader when it returns.

**Risks:**
- Promoting a lagging follower loses acknowledged writes
- Split brain, with two leaders accepting writes
- Detection timeouts: too short causes false failovers, too long extends downtime

## Distributed transactions

When one business action touches several services or databases.

| | Two-phase commit (2PC) | Saga |
|---|---|---|
| How | Coordinator asks all to prepare, then commit | Sequence of local transactions with compensating actions |
| Consistency | Atomic | Eventual |
| Availability | Blocks if coordinator fails | High |
| Complexity | Locking, tight coupling | Must design compensations |
| Use when | Few systems, same trust domain | Microservices, long running flows |

**Saga example, order flow:** reserve inventory, charge payment, create shipment. If payment fails, run the compensation: release inventory.

Two styles: **orchestration** (a central coordinator drives steps) and **choreography** (services react to events). Orchestration is easier to reason about, choreography is more decoupled.

Pair sagas with the **outbox pattern** so a database write and its event publish cannot diverge, and with [idempotency](05-reliability-and-protection.md) so retries are safe.

## Interview questions

1. Is your design CP or AP, and what does the user see during a partition?
2. A user updates their profile and sees the old value. Why, and how do you fix it?
3. Explain Saga vs 2PC for a checkout flow.
4. How does a system avoid split brain after a failover?
