---
title: "The stack — Go, Postgres, and pgvector"
description: "The technology behind Group Scout and why these choices fit a background data pipeline."
pubDate: "2026-04-12"
draft: true
---

I want to explain my technology choices. The _why_ is more interesting than the stack itself.

Here's what Group Scout is built on:

| Layer          | Technology                  | Why                                                     |
| -------------- | --------------------------- | ------------------------------------------------------- |
| Language       | Go                          | Background pipelines are Go's natural habitat           |
| Database       | PostgreSQL + pgvector       | Production stability, UUIDs, and vector support for RAG |
| AI enrichment  | Claude Haiku (Anthropic)    | ~$0.001 per permit, fast, great at structured JSON      |
| Infrastructure | Docker Compose              | Reproducible environments for the app, DB, and n8n      |
| Notifications  | Slack incoming webhook      | No app setup, Block Kit makes rich cards easy           |
| Monitoring     | Sentry + structured logging | Production-grade error tracking and observability       |

No framework. No ORM. No generated code. Just `database/sql`, `net/http`, and the standard library.

---

## Why Go

I know Go well, and it is the right tool for background data pipelines. Goroutines simplify concurrent HTTP calls. The type system catches compile-time mistakes that would be runtime bugs in Python or JavaScript. The binary compiles to a single executable.

The codebase uses interfaces, which helps manage multiple data sources. The `Collector` interface is:

```go
type Collector interface {
    Name() string
    Collect(ctx context.Context) ([]RawProject, error)
}
```

Every data source implements this interface. Building permits use one implementation; contract awards use another. Adding a third source doesn't change the pipeline; I just write a new struct. Go makes this extensibility clean.

---

## Why PostgreSQL (and why pgvector)

I started with SQLite, which was perfect for the first few weeks. As the system grew, I needed more. I moved to PostgreSQL (using `pgvector/pgvector:pg17`) for:

1. **Native UUIDs:** No string-to-ID mapping.
2. **JSONB support:** Indexing and querying raw project data.
3. **pgvector:** Storing lead embeddings enables similarity searches to find "leads like this one" or match against historical "won" business.

I used `golang-migrate` for migrations and `github.com/jackc/pgx/v5` as the driver. It is pure Go and maintains a simple build process.

---

## Why Docker Compose

Docker Compose handles orchestration for the Group Scout app, Postgres, n8n, Prometheus, and Grafana. `docker compose up -d` makes the ecosystem live.

---

## Why Claude Haiku for enrichment

A building permit omits:

- Crew type (local or fly-in)
- Crew size
- Project duration
- Outreach timing

A human infers these from project type, value, location, and contractor. Language models excel here.

The interesting part here isn’t learning how to use an LLM — it’s deciding where it earns its keep, and where the system should stay deterministic.

I use Claude Haiku. Enrichment costs about $0.001 per permit, or pennies per month. Haiku is fast, taking about a second per permit. Sonnet is more capable but slower and 5x the cost; Haiku does the job.

I chose the smaller model deliberately. Most of the system should remain deterministic.

The model returns structured JSON: priority score, project type, crew size, duration, out-of-town likelihood, outreach timing, and notes. Predictable and actionable.

---

## Why Slack

My wife already has Slack on her phone. A Monday morning message delivers this week's leads.

Slack's Block Kit enables rich cards with score badges, priority indicators, and PDF links without frontend code. The "UI" is a webhook POST with structured JSON.

I want useful output with minimum friction. Slack nails that.

---

## What's not in the stack

No web framework, ORM, or message queue. Just Docker, Postgres, and Go.

This weekend project solves a real problem. Every dependency requires maintenance. The standard library does much. Start there.

Next: building the storage layer.
