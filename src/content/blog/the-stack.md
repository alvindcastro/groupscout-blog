---
title: "The stack — Go, Postgres, and pgvector"
description: "The technology behind Group Scout and why these choices fit a background data pipeline."
pubDate: "2026-04-19"
draft: false
---

I will explain my technology choices; the reasoning is more interesting than the stack.

Group Scout uses these technologies:

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

I know Go well. It suits background data pipelines. Goroutines simplify concurrent calls. The type system catches mistakes that would be runtime bugs in Python or JavaScript. The binary compiles to a single executable.

The codebase uses interfaces to manage multiple data sources. The `Collector` interface is:

```go
type Collector interface {
    Name() string
    Collect(ctx context.Context) ([]RawProject, error)
}
```

Every data source implements this interface. Building permits use one implementation; contract awards use another. Adding a third source requires only a new struct, not a change to the pipeline. Go makes this extensibility clean.

---

## Why PostgreSQL (and why pgvector)

I started with SQLite, which served for the first few weeks. As the system grew, I moved to PostgreSQL (using `pgvector/pgvector:pg17`) for:

1. **Native UUIDs:** No string-to-ID mapping.
2. **JSONB support:** Indexing and querying raw project data.
3. **pgvector:** Storing lead embeddings enables similarity searches to find "leads like this one" or match against historical "won" business.

I use `golang-migrate` for migrations and `pgx` as the driver. Both are pure Go and maintain a simple build process.

---

## Why Docker Compose

Docker Compose orchestrates the app, Postgres, n8n, Prometheus, and Grafana. `docker compose up -d` makes the ecosystem live.

---

## Why Claude Haiku for enrichment

A building permit omits:

- Crew type (local or fly-in)
- Crew size
- Project duration
- Outreach timing

A human infers these from project type, value, location, and contractor. Language models excel here.

The challenge is deciding where an LLM earns its keep and where the system should remain deterministic.

I use Claude Haiku. Enrichment costs $0.001 per permit. Haiku is fast; it takes one second per permit. Sonnet is more capable but slower and more expensive; Haiku suffices.

I chose the smaller model deliberately. Most of the system should remain deterministic.

The model returns structured JSON: priority, type, crew size, duration, likelihood, timing, and notes. Predictable and actionable.

---

## Why Slack

My wife has Slack on her phone; a Monday morning message delivers the leads.

Slack's Block Kit enables rich cards without frontend code. The UI is a webhook POST with structured JSON.

I want useful output with minimum friction. Slack provides it.

---

## What's not in the stack

No web framework, ORM, or message queue. Just Docker, Postgres, and Go.

This project solves a real problem. Every dependency requires maintenance; the standard library suffices. Start there.

Next: the storage layer.
