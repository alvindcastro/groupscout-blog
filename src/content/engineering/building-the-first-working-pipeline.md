---
title: "Building the First Working Pipeline"
description: "How the collector interface, deduplication, Claude enrichment, and Slack notification came together into a pipeline that could actually find a lead."
pubDate: "2026-04-04"
tags: ["go", "architecture", "claude", "slack", "sqlite"]
draft: false
---

The goal was simple: collect a building permit, score it, and send a Slack message. I used one collector, one enrichment call, and one notification.

It took longer than expected; the necessary abstractions did not exist.

---

## Schema First

The pipeline required storage. Without deduplication, the enrichment layer would call Claude on the same permit every time the pipeline ran.

Two tables mattered at the start.

**`raw_projects`** stores what each collector returns. Each row includes a SHA-256 hash of the payload. This hash is the deduplication key: before enriching a project, the pipeline checks if the hash exists. If it does, the project is skipped.

**`leads`** stores enriched records. A lead is a project that passed the pre-scorer and went to Claude. It holds extracted fields—type, crew size, duration, score—alongside the rationale and outreach draft.

The schema initially used SQLite. The choice was pragmatic: it required no infrastructure and was fast enough for weekly permit volumes. The migration to Postgres occurred later once the pipeline was stable.

---

## The First Collector

The first data source was Richmond, BC's weekly permit report. The collector downloads the PDF, extracts text, and parses permits.

Every collector in the system implements the same interface:

```go
type Collector interface {
    Collect(ctx context.Context) ([]RawProject, error)
}
```

The interface is narrow. A collector's only job is to fetch data and return `RawProject` structs. It knows nothing of storage or notifications. Adding a new source requires only one new method.

`RawProject` is the normalization layer. Every source produces different data; `RawProject` flattens this into a common shape before storage.

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

`RawData` holds source-specific information. The pipeline ignores it; Claude inspects it.

---

## The Enrichment Call

After deduplication, qualifying projects go to the enrichment layer. The Claude call asked three questions:

1. Does this project likely require out-of-town workers?
2. How many workers, and for how long?
3. What is the priority score from 1 to 10?

The response was structured JSON. The prompt forbade markdown and commentary. A parse failure meant the lead was skipped. This occurred frequently during testing, so I made the prompt more directive.

One early decision: the prompt told Claude to score based on hotel demand, not project value. A $2M commercial build near an airport outranks a $5M residential tower downtown. Scoring reflects group lodging demand.

---

## The First Notification

High-scoring leads triggered Slack messages. The format was minimal: name, value, location, score, and rationale. It provided enough information for a sales manager to decide whether to call.

The rationale was the most useful part. A score of eight means nothing without an explanation of crew size, timeline, and location. That sentence makes the tool valuable.

Instant alerts fired for scores of nine or higher; everything else went into the Monday digest.

---

## What This Established

After the first version, the system could process a permit, evaluate it, and notify a human. The code was six hundred lines.

The contracts were more important than the code. The `Collector` interface and `RawProject` struct remained unchanged as I added sources. Each new collector plugged into the pipeline without affecting enrichment or notification.

The pipeline scaled because its foundation was narrow and correct.
