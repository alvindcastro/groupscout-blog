---
title: 'The stack — Go, SQLite, and a $0.001 AI call'
description: 'The technology choices behind groupscout, and why each one made sense for a background data pipeline built by one person on weekends.'
pubDate: '2026-03-29'
heroImage: '../../assets/blog-placeholder-3.jpg'
---

One of the things I wanted to do with this blog is actually explain the technology choices — not just list them, but explain *why*. Because the choices are usually more interesting than the stack itself.

Here's what groupscout is built on:

| Layer | Technology | Why |
|---|---|---|
| Language | Go | Background pipelines are Go's natural habitat |
| Database | SQLite | Zero ops, runs anywhere, no C compiler headaches |
| AI enrichment | Claude Haiku (Anthropic) | ~$0.001 per permit, fast, great at structured JSON |
| Notifications | Slack incoming webhook | No app setup, Block Kit makes rich cards easy |
| Config | `.env` + environment variables | 12-factor: no secrets ever touch the code |

No framework. No ORM. No generated code. Just `database/sql`, `net/http`, and the standard library.

---

## Why Go

I know Go well. That's the honest answer. But it's also genuinely the right tool for this job.

Go is excellent at background data pipelines. Goroutines make concurrent HTTP calls easy. The type system catches mistakes at compile time that would surface as runtime bugs in Python or JavaScript. The binary compiles to a single executable you can run anywhere — no runtime, no interpreter, no dependency hell.

The codebase is also naturally organized around interfaces, which matters a lot for a pipeline with multiple data sources. The `Collector` interface looks like this:

```go
type Collector interface {
    Name() string
    Collect(ctx context.Context) ([]RawProject, error)
}
```

Every data source implements that interface. Richmond building permits: one implementation. BC Bid contract awards: another implementation. When I add a third source, the pipeline doesn't change — I just write a new struct that satisfies the interface. That kind of clean extensibility is easy to achieve in Go.

---

## Why SQLite (and why not the obvious library)

SQLite for local development is a no-brainer — zero setup, single file, runs anywhere, good enough for the data volumes here (tens to hundreds of leads per week, not millions).

The interesting choice is *which* SQLite library. The obvious one is `mattn/go-sqlite3` — it's been around forever, widely used, battle-tested. The problem: it requires CGO, which means you need a C compiler on your machine.

On a Mac, that's usually fine. On Windows (which is my dev machine), it's a mess. CGO on Windows requires MinGW or a specific GCC setup. I've been burned by this before. An hour of toolchain wrestling to write a database query is not a good use of a Saturday morning.

The alternative is `modernc.org/sqlite` — a port of SQLite to pure Go. No C compiler. No CGO. Same `database/sql` interface. The only tradeoff is slightly slower performance, which does not matter at all for this use case.

`modernc` it is.

---

## Why Claude Haiku for enrichment

A building permit tells you: there's a $1.2M warehouse being built at this address by this applicant. It doesn't tell you:
- Whether the crew will be local or fly-in
- How large the crew is likely to be
- How long the project will run
- When to reach out

A human reading the permit can make reasonable inferences based on project type, value, location, and the contractor's name. That's exactly what language models are good at.

I'm using Claude Haiku (Anthropic's fastest, cheapest model) for this. The cost per permit is approximately $0.001 — less than a tenth of a cent. The pipeline runs weekly on maybe 5–10 filtered permits. The entire enrichment step costs pennies per month.

Haiku is also fast. The enrichment call takes about a second per permit. Sonnet would be more capable but slower and 5x the cost for a use case where Haiku is already doing the job well.

The model returns structured JSON: priority score, project type, estimated crew size, duration, out-of-town likelihood, outreach timing suggestion, notes. Predictable, parseable, actionable.

---

## Why Slack

The simplest possible answer: my wife already has Slack on her phone.

No app to build. No login to manage. No dashboard that'll be ignored after the first week. Just a message that shows up Monday morning with this week's leads.

Slack's Block Kit format is also genuinely good for rich card layouts. Score badges, priority indicators, direct links to the source PDF — all of it is possible without writing any frontend code. The "UI" is just a webhook POST with some structured JSON.

The goal was always: useful output, minimum friction. A Slack message nails that.

---

## What's not in the stack

No web framework. No ORM. No message queue. No Docker (yet). No external dependencies beyond what's needed.

This is a weekend project built to solve a real problem. Every dependency you add is a dependency you have to maintain, update, and debug. The standard library does a lot. Start there.

Next post: building the storage layer — which, counterintuitively, is where I started before writing a single line of scraping code.
