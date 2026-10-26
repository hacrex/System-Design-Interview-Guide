# 00 Interview Framework

Use the same seven steps for every problem. In a 45 minute round, roughly follow the time budget shown.

| Step | Goal | Time |
|------|------|------|
| 1. Clarify requirements | Agree on scope | 5 min |
| 2. Estimate scale | Size the problem | 3 min |
| 3. Define the API | Pin down the contract | 5 min |
| 4. Model the data | Choose storage shape | 5 min |
| 5. High-level design | Boxes and arrows | 8 min |
| 6. Deep dive | Go deep on 1 or 2 hard parts | 12 min |
| 7. Bottlenecks and tradeoffs | Failure, scale, cost | 7 min |

## Step 1: Clarify requirements

Ask before you draw.

- **Functional:** What must the system do? Name the top 3 features and explicitly cut the rest.
- **Non-functional:** Latency target, availability target, consistency needs, durability, read vs write ratio.
- **Constraints:** Users, regions, mobile or web, existing systems, budget.

Say the scope back in one sentence so the interviewer can correct you.

## Step 2: Estimate scale

Turn users into numbers: QPS, storage per year, bandwidth, cache size. See [06 Estimation](06-estimation-and-metrics.md). Round aggressively. The point is to decide whether one machine is enough, not to be exact.

## Step 3: Define the API

List the main endpoints or RPCs with inputs and outputs. Mention pagination, authentication and idempotency for writes. This forces clarity before architecture.

## Step 4: Model the data

- What are the entities and their relationships?
- What are the access patterns? Design storage around reads and writes you listed, not around tables you like.
- SQL or NoSQL, and why. See [02 Data storage](02-data-storage.md).

## Step 5: High-level design

Draw the simplest design that meets the requirements: client, load balancer, services, cache, database, queue. Walk one read request and one write request end to end. Start simple, then scale.

## Step 6: Deep dive

Pick the one or two parts that are hardest for this problem (for a URL shortener: ID generation; for a feed: fan-out strategy). Let the interviewer steer. Show the options, the tradeoff, and your choice.

## Step 7: Bottlenecks and tradeoffs

Walk the design looking for:

- Single points of failure
- Hot keys and hot partitions
- What happens when a dependency is slow or down
- Data growth in year 3
- Cost, operations and monitoring

## Answer pattern for any tradeoff

State the options, the criteria that matter here, your pick, and what you give up.

> "I would use X because the workload is read heavy and we can tolerate stale data for a few seconds. The cost is that users may briefly see old values, which is acceptable for this feature."

## Common mistakes

- Jumping to boxes before clarifying requirements
- Naming technologies without saying why
- Designing for Google scale when the numbers say otherwise
- Ignoring failure modes
- Talking for 10 minutes without checking in with the interviewer

## Practice prompts

1. Take any app you use daily and run steps 1 to 3 in five minutes.
2. Record yourself explaining a design. Count how many times you gave a reason.
