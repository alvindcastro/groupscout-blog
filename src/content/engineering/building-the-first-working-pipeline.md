---
title: 'Building the First Working Pipeline'
description: 'How the collector interface, deduplication, Claude enrichment, and Slack notification came together into a pipeline that could actually find a lead.'
pubDate: '2026-04-12'
tags: ['go', 'architecture', 'claude', 'slack', 'sqlite']
---

The goal was simple: collect a building permit, score it, and send a Slack message. Nothing else. No multi-source pipeline, no production infrastructure. Just one collector, one enrichment call, one notification.

It took longer than expected — not because the code was hard, but because the right abstractions didn't exist yet.

---

## Schema First

The pipeline couldn't work without somewhere to put data. More importantly, without deduplication, the enrichment layer would call Claude on the same permit every time the pipeline ran.

Two tables mattered at the start.

**`raw_projects`** stores what each collector returns before any enrichment. Every row includes a SHA-256 hash of the raw payload. That hash is the deduplication key — before enriching a project, the pipeline checks whether the hash already exists. If it does, the project is skipped. No API call, no re-processing.

**`leads`** stores enriched records. A lead is a `raw_project` that passed the pre-scorer and was sent to Claude. It holds the extracted fields (project type, estimated crew size, estimated duration, priority score) alongside the AI-generated rationale and outreach draft.

The schema used SQLite initially. The choice was pragmatic — no infrastructure to set up, easy to inspect with any SQL client, fast enough for weekly permit volumes. The migration to Postgres came later once the pipeline was stable and the data model had stopped changing.

---

## The First Collector

The first data source was Richmond, BC's weekly building permit report — a PDF published on a municipal website. The collector downloads it, extracts text with `pdftotext`, and parses permit records from the output.

Every collector in the system implements the same interface:

```go
type Collector interface {
    Collect(ctx context.Context) ([]RawProject, error)
}
```

The interface is intentionally narrow. A collector's only job is to fetch data from one source and return a slice of `RawProject` structs. It knows nothing about storage, scoring, or notifications. This boundary matters — adding a new source means writing one new `Collect` method, not touching the pipeline.

`RawProject` is the normalization layer. Every source — permits, RSS feeds, bid databases, event scrapers — produces structurally different data. `RawProject` flattens this into a common shape before anything hits storage:

```go
type RawProject struct {
    SourceID    string
    Source      string
    Title       string
    Description string
    Value       float64
    Location    string
    RawData     map[string]any
}
```

`RawData` holds anything source-specific. The pipeline doesn't inspect it. Claude does.

---

## The Enrichment Call

After deduplication, qualifying projects go to the enrichment layer. The Claude call took a `RawProject` and asked three questions:

1. Does this project likely require out-of-town workers?
2. How many workers, and for how long?
3. What is the priority score from 1 to 10?

The response came back as structured JSON. The prompt was explicit about format — no markdown, no commentary, just the fields. A parse failure meant the lead was skipped and the error was logged. This happened more often than expected during early testing, which pushed the prompt toward being more directive about output shape.

One early decision: the prompt told Claude to score based on hotel demand potential, not project value. A $2M commercial build near an airport scores higher than a $5M residential tower downtown. The scoring isn't about how big the project is — it's about whether group lodging demand follows it.

---

## The First Notification

High-scoring leads triggered a Slack message. The initial format was minimal — project name, value, location, score, and the plain-English rationale Claude generated. Nothing interactive. Just enough for a sales manager to decide whether to make a call.

The rationale sentence was the most useful part from day one. A score of 8 means nothing without "Construction crew of 40–60 workers, 8-month project timeline, 3km from hotel, workers likely from out-of-province." That sentence is what makes the tool worth using.

Instant alerts fired for scores ≥ 9. Everything else went into the weekly Monday digest.

---

## What This Established

After this first working version, the system could do one thing end-to-end: take a Richmond permit, decide whether it mattered, and tell a human about it. The code was about 600 lines across five files.

More important than the code were the contracts. The `Collector` interface and `RawProject` struct never changed as new sources were added — BC Bid, the Vancouver Convention Centre, Eventbrite, Google News, government infrastructure announcements. Each new collector plugged into the same pipeline without touching enrichment or notification.

The pipeline scaled because the foundation was narrow enough to be right.
