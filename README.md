# System Design Interview Prep: Step by Step

A practical, fundamentals-first guide to system design interviews. Built around 25 core concepts, one repeatable interview framework, and worked case studies you can adapt.

Fundamentals matter more than ever. Tools and company names change, but the tradeoffs behind them do not.

## Who this is for

- Engineers preparing for senior system design rounds at product companies
- Cloud, DevOps and backend engineers who build systems daily but want a structured way to explain them under time pressure
- Anyone who wants one place to revise before an interview

## How to use this repo

1. Read the [interview framework](docs/00-interview-framework.md) first. It is the skeleton for every answer.
2. Work through the modules in order. Each one ends with interview questions to answer out loud.
3. Do the case studies. Try the problem yourself using the [blank template](templates/design-template.md) before reading the solution.
4. Follow the [8-week plan](study-plan/8-week-plan.md) if you want a schedule.

## Learning path

Tick these off as you go.

### Phase 1: Method
- [ ] [00 Interview framework](docs/00-interview-framework.md)

### Phase 2: Building blocks
- [ ] [01 Scaling and traffic](docs/01-scaling-and-traffic.md): scalability, load balancing, caching
- [ ] [02 Data storage](docs/02-data-storage.md): SQL vs NoSQL, indexing, replication, sharding, partitioning, consistent hashing
- [ ] [03 Consistency and distributed data](docs/03-consistency-and-distributed-data.md): CAP, consistency models, replication lag and failover, distributed transactions
- [ ] [04 Communication](docs/04-communication.md): queues, pub/sub, API design, API gateway, microservices vs monolith

### Phase 3: Hard problems and applied skills
- [ ] [05 Reliability and protection](docs/05-reliability-and-protection.md): rate limiting, idempotency, availability, failure handling
- [ ] [06 Estimation and performance metrics](docs/06-estimation-and-metrics.md)
- [ ] [07 Observability and security](docs/07-observability-and-security.md)

### Phase 4: Practice
- [ ] [Case study: URL shortener](case-studies/url-shortener.md)
- [ ] [Case study: Rate limiter](case-studies/rate-limiter.md)
- [ ] [Case study: Key-value store](case-studies/key-value-store.md)
- [ ] [Case study: Notification system](case-studies/notification-system.md)
- [ ] More problems listed in [case-studies/README.md](case-studies/README.md)

Module numbers flow 00 → 07 across the phases. The order is intentional: method first, then the building blocks, then the harder operational topics, and finally hands-on practice.

## The 25 concepts and where they live

The full concept-to-module map lives in the [cheat sheet](docs/cheat-sheet.md#the-25-concepts-at-a-glance).

Extra topics added beyond the list because senior interviews probe them: quorum and leader election, hot keys, backpressure, retries with backoff, circuit breakers. They live in modules 03 and 05.

## Repo structure

```
system-design-interview-guide/
  README.md
  LICENSE
  docs/                  concept modules and the interview framework
  case-studies/          worked designs and a list of practice problems
  templates/             blank design template to practice with
  study-plan/            8-week schedule
```

Quick reference while you prep: [docs/cheat-sheet.md](docs/cheat-sheet.md) maps interview signals to modules and defines the extra topics (quorum, leader election, hot keys, backpressure, circuit breakers).

Visual references: [docs/diagrams.md](docs/diagrams.md) has Mermaid diagrams for load balancing, replication, consistent hashing, quorum, leader election, circuit breakers, rate limiting, and more.


## Contributing

Issues and pull requests are welcome. Good contributions: corrections, new case studies, better diagrams, real-world failure stories. Keep entries concise, comparison-oriented and practical.

Contributors are acknowledged in [CONTRIBUTORS.md](CONTRIBUTORS.md).

## License

MIT. See [LICENSE](LICENSE).
