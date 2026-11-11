# Changelog

All notable changes to the System Design Interview Guide are documented here.

## [Unreleased]

### Added
- `docs/failure-stories/` — real-world outage write-ups (AWS S3 2017, Knight Capital 2012, Cloudflare 2022) with design lessons and module cross-references
- `docs/common-mistakes.md` — 15 common interview anti-patterns and what to do instead
- `docs/behavioral-resume-prep.md` — STAR stories, leadership signals, and resume tips for senior system design loops
- Mermaid topology diagrams to all 7 newer case studies (chat system, distributed cache, file storage & sync, search autocomplete, web crawler, ride hailing, metrics & monitoring)
- Mermaid topology diagrams to the 3 original case studies (URL shortener, rate limiter, key-value store)

### Changed
- `README.md` — added "More resources" section linking failure stories, common mistakes, behavioral prep, and cram plan
- `docs/cheat-sheet.md` — extended "How to use" with links to failure stories, common mistakes, cram plan, and behavioral prep

## [v1.2.0] — 2026-11-11

### Added
- Solved case studies: chat system, distributed cache, file storage & sync, search autocomplete, web crawler, ride hailing, metrics & monitoring
- All 14 practice problems now solved and linked from `README.md` and `case-studies/README.md`

## [v1.1.0] — 2026-11-11

### Added
- `CODE_OF_CONDUCT.md`, `SECURITY.md`, `.gitignore`, `.github/PULL_REQUEST_TEMPLATE.md`
- `.github/workflows/docs.yml` — GitHub Actions CI with markdownlint and markdown-link-check
- `docs/algorithms-reference.md` — token bucket, consistent hashing, LRU, exponential backoff, leader election, circuit breaker
- `docs/mock-interview-bank.md` — phased 45-minute mock interview prompts
- `docs/senior-staff-topics.md` — multi-region, cost, blast radius, observability, incident judgment
- `study-plan/cram-plan.md` — 72h / 48h / 24h / 4h short-notice plans
- `docs/failure-stories/` (real-world outage write-ups)
- `docs/common-mistakes.md`, `docs/behavioral-resume-prep.md`
- Mermaid diagrams to 7 new case studies
- Table of contents to modules 03, 04, 05
- `CONTRIBUTORS.md`
- Deduplicated "25 concepts" table (canonical in `docs/cheat-sheet.md`)

### Changed
- `README.md` — linked new case studies in learning path; added CONTRIBUTORS reference
- `case-studies/README.md` — marked notification system, news feed, payment system, video streaming as Solved
- `docs/cheat-sheet.md` — linked algorithms reference and mock interview bank

## [v1.0.0] — 2026-11-11

### Added
- Initial release: 8 concept modules, 3 solved case studies (URL shortener, rate limiter, key-value store), 8-week study plan, design template, grading rubric, cheat sheet, Mermaid diagrams, estimation examples, contributing guidelines, issue templates
