# Contributing

Issues and pull requests are welcome. Good contributions: corrections, new case studies, better diagrams, real-world failure stories. Keep entries concise, comparison-oriented and practical.

## How to contribute

1. Fork the repo and create a branch from `main`.
2. Make your change (see the checklists below).
3. Open a pull request against `main`. Describe what changed and why.

## Case-study PR checklist

If you are adding or editing a case study, make sure it:

- [ ] Copies [`templates/design-template.md`](templates/design-template.md) and fills in all seven framework sections
- [ ] Includes numbers in the estimation (QPS, storage per year, bandwidth, cache size)
- [ ] Has one deep dive with at least two options compared and a stated choice with tradeoffs
- [ ] Lists bottlenecks and tradeoffs (SPOF, hot keys, failure behavior, year 3 growth, monitoring)
- [ ] Ends with follow-up questions to practice
- [ ] Is linked from [`case-studies/README.md`](case-studies/README.md) (table + suggested order) and from the main [`README.md`](README.md) learning path
- [ ] Is marked as Solved or Practice in the case-studies table

Score your draft with [`docs/grading-rubric.md`](docs/grading-rubric.md) before opening the PR — aim for no 0s.

## Other contribution types

- **Corrections:** fix a typo, stale number, or broken link. Link the issue if you open one.
- **Diagrams:** Mermaid diagrams render natively on GitHub and work well in modules 01–03.
- **Real-world failure stories:** a short write-up of an actual outage or incident, what failed, and what the system design lesson is.

## Style

- Concise and comparison-oriented: show options, tradeoffs, and a choice — not just "the best tool."
- Prefer tables and ASCII/Mermaid diagrams over long prose.
- Every technology choice should have a reason.
