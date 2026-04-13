---
title: "Where it stands — Production-ready and scaling"
description: "The pipeline is a production-grade system with multiple sources, structured monitoring, and a Postgres backend."
pubDate: "2026-04-12"
draft: true
---

> This is a snapshot of the system as it stands in late March 2026.

The pipeline runs with one command:

```bash
go run ./cmd/server/ --run-once
```

What happens:

1. Scrapes the weekly reports for the latest PDF URL.
2. Downloads the PDF, extracts text, and parses permits.
3. Filters commercial projects above the threshold.
4. Checks permits against the database and skips seen items.
5. Calls Claude to enrich permits.
6. Posts a Block Kit digest to Slack.

In a recent run, three of twenty permits passed the filter and reached the enrichment stage. The code is wired and tested.

**Test coverage:** thirty-two unit tests cover the collector, enrichment, and notification packages. Every pure function is tested.

---

## What I learned

**PDF parsing is complex.** Every library has encoding edge cases. Use `pdftotext` rather than fighting a pure Go library.

**Content-aware parsing beats positional parsing.** PDF formats shift. Detecting fields by appearance is more resilient than counting lines.

**Deduplicate first, enrich second.** Persisting the raw record before calling Claude ensures that a failed enrichment does not lose data.

**Claude Haiku excels at structured extraction.** It costs little and produces actionable output; the system prompt makes a difference.

**Build the storage layer before the scraper.** The schema clarifies the data's shape; everything upstream then aligns with it.

---

## What's next

### Phase 16 — LLM Provider Abstraction

- Abstract LLM calls
- Support OpenAI, Groq, and local Ollama
- Fallback logic: if Claude fails, retry with GPT-4o-mini

### Phase 17 — Airport Disruption Alert System

- A service to monitor airports for cancellations and weather alerts
- Real-time Slack alerts
- Integration with hotel inventory

### Phase 18 — Contact Enrichment

- Identify decision-makers using Hunter.io
- Attach contact details to leads in Slack

---

## The honest update

The pipeline works and solves a problem. My wife is cautiously optimistic; so am I.

The code is on GitHub. Next: the scheduler.
