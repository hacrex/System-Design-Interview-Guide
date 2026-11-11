# Case Study: Payment System

Try it yourself first using the [blank template](../templates/design-template.md). Then compare.

## 1. Requirements

**Functional**
- A user can send money to another user (payment)
- A user can request a refund for a payment
- View payment history and current balance
- Idempotent payments: retrying a payment must not charge twice

**Non-functional**
- Payments must be exactly-once processed — no double charges, no lost money
- Strong consistency for balances (you cannot spend money you don't have)
- High availability: payments must keep working during a single zone failure
- Durability: an acknowledged payment survives a crash
- Low latency: payment authorization under ~200 ms at p99

Out of scope for this round: KYC/AML verification, fraud detection models, multi-currency/FX, settlement to bank accounts, dispute chargebacks.

## 2. Estimation

- 50 million DAU, each user makes 2 payments per day on average
- 100M payments/day → ~1,200 writes/s average, ~5,000/s peak
- Each payment record: ~200 bytes (payer, payee, amount, currency, status, idempotency key, timestamp)
- Daily storage: 100M × 200 bytes ≈ 20 GB/day; ~7 TB/year
- Balance reads: ~3 per payment → ~300M reads/day → ~3,500 reads/s average

Conclusion: write volume is modest but correctness is critical. Use a single strongly-consistent ledger database with sharding by account, plus idempotency keys and a saga for cross-account money movement.

## 3. API

```
POST   /v1/payments
       body: {
         "idempotency_key": "...",
         "payer_account_id": "...",
         "payee_account_id": "...",
         "amount": 1000,
         "currency": "USD",
         "description": "optional"
       }
       returns: { "payment_id": "...", "status": "authorized|rejected", "created_at": "..." }

POST   /v1/refunds
       body: {
         "idempotency_key": "...",
         "payment_id": "...",
         "amount": 500
       }
       returns: { "refund_id": "...", "status": "authorized|rejected" }

GET    /v1/accounts/{account_id}/balance
       returns: { "account_id": "...", "balance": 5000, "currency": "USD" }

GET    /v1/accounts/{account_id}/payments?limit=20&cursor=...
       returns: list of payments with cursor
```

Idempotency key: client-generated UUID. Reusing a key with a different amount must be rejected; reusing with the same payload returns the stored result.

## 4. Data model

Access patterns:
- Debit/credit an account balance (strongly consistent, write-heavy)
- Insert a payment ledger entry (append-only)
- Fetch an account's payment history (read-heavy, latest first)
- Check idempotency key before processing (unique lookup)

| Entity | Field | Notes |
|--------|-------|-------|
| Account | account_id (PK) | Unique account identifier |
| Account | balance | Current balance, strongly consistent |
| Account | currency | Account currency |
| Payment | payment_id (PK) | Snowflake ID or UUID |
| Payment | idempotency_key | Unique, indexed, checked before processing |
| Payment | payer_account_id | Indexed |
| Payment | payee_account_id | Indexed |
| Payment | amount, currency | Money amount |
| Payment | status | pending, authorized, rejected, failed |
| Payment | created_at | Sort key for history |

Storage choice: **relational database (PostgreSQL)** sharded by `account_id`. Why: payments need ACID transactions (debit payer + credit payee atomically), strong consistency, and an append-only audit ledger. NoSQL sacrifices the atomic multi-row transaction we need here. A write-ahead log gives durability.

## 5. High-level design

```
Client -> API Gateway (idempotency check) -> Payment Service
                                               |
                                    +----------+----------+
                                    |                     |
                            Ledger DB (shard by account)  Outbox Table
                                    |                     |
                                    |              Message Relay -> Kafka
                                               |
                                       Reconciliation Worker
```

**Write path (payment):**
1. Client sends payment with idempotency key.
2. Gateway checks idempotency key; if seen, return cached result.
3. Payment Service runs in one DB transaction: check payer balance ≥ amount, debit payer, credit payee, insert payment ledger row as `authorized`, record idempotency key, write an outbox event.
4. Commit → return `payment_id` and status.
5. Message Relay publishes the outbox event to Kafka for downstream (notifications, email receipts, analytics).

**Read path (balance/history):** Query the shard for the account's balance and payment history.

**Failure during write:** If the transaction fails (insufficient funds, deadlock, constraint violation), roll back and return `rejected`. Never partially apply.

## 6. Deep dive: saga for cross-service money movement

When a payment touches more than the ledger (e.g., notify a fraud service, send an email, update a rewards ledger), a single DB transaction cannot span all services. Use a saga.

| Approach | How it works | Pros | Cons |
|----------|--------------|------|------|
| 2PC (Two-phase commit) | Coordinator asks all participants to prepare, then commits or aborts all. | Atomic across services | Blocks if coordinator fails; tightly coupled; poor availability |
| Saga (orchestration) | Central orchestrator runs local transactions in order; on failure, runs compensating transactions in reverse. | High availability, decoupled | Eventual consistency; compensations must be designed |
| Saga (choreography) | Each service emits events; next service reacts. No central coordinator. | Most decoupled | Harder to reason about and debug |

**Recommended:** Orchestration-based saga.

Payment saga flow:
1. **Reserve funds** in payer's ledger (local transaction: mark amount as reserved).
2. **Credit payee** (local transaction).
3. **Release reservation / finalize** (mark payment authorized).

If step 2 fails (payee account frozen):
- Compensation: **release reserved funds** back to payer.
- Mark payment `rejected`.

Each saga step is idempotent (keyed by payment_id), so if the orchestrator retries a step after a timeout, it doesn't double-apply.

**Outbox pattern:** The ledger write and the saga event publish happen in the same DB transaction, so they cannot diverge. A relay reads the outbox table and publishes to Kafka.

## 7. Bottlenecks and tradeoffs

- **Hot account:** A popular account (e.g., a merchant) receives many concurrent credits. Shard by account_id so all mutations for one account go to the same shard; the shard serializes them. Accept short queueing at the shard.
- **Deadlocks:** Two concurrent transfers between the same pair of accounts can deadlock if locks are taken in different orders. Always lock accounts in a consistent order (e.g., lower account_id first).
- **Strong consistency cost:** Synchronous replication within a region means a zone failure can make a shard temporarily unavailable for writes. Accept brief unavailability over silent balance divergence — for money, consistency wins over availability during a partition (CP).
- **Reconciliation:** Run a nightly job that recomputes each account's balance from the append-only ledger and compares it to the stored balance. Alert on any mismatch.
- **Idempotency key storage:** Keep keys for a bounded window (e.g., 24–48 hours) and reject reused keys with different payloads.
- **Year 3 growth:** 7 TB/year of ledger data; archive settled payments older than 90 days to cold storage, keep recent payments hot. Shard count grows with account count.
- **Monitoring and alerts:** Track payment success/failure rates, ledger transaction latency (p99), double-charge attempts (idempotency key reuse with different amounts), reconciliation mismatches, and outbox lag. Page on any mismatch or spike in rejected payments.

## Follow-up questions to practice

1. How do you handle a partial failure where the payer was debited but the payee was not credited?
2. What changes if payments can be in multiple currencies?
3. How do you prevent a malicious user from making thousands of tiny payments to exhaust the ledger?
4. How do you reconcile balances after a region failover?
5. When would you choose 2PC over a saga for payments, and why is that usually wrong?
