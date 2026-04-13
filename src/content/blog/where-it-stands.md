---
title: "Where it stands — Production-ready and scaling"
description: "The pipeline is a production-grade system with multiple sources, structured monitoring, and a Postgres backend."
pubDate: "2026-04-12"
draft: true
---

> This is a snapshot of the system as it stands in late March 2026.

The pipeline runs. One command:

```bash
go run ./cmd/server/ --run-once
```

What happens:

1. Scrapes the weekly reports page for the latest PDF URL.
2. Downloads the PDF, extracts text with `pdftotext`, and parses permit records.
3. Filters commercial sub-types above the value threshold.
4. Checks each permit against the DB and skips seen items.
5. Calls Claude to enrich new permits into scored leads.
6. Posts a Block Kit digest to Slack.

On a recent run, 3 of 20 permits passed the filter. All three hit the enrichment stage. The pipeline code is fully wired and tested.

**Test coverage:** 32 unit tests cover the collector, enrichment, and notify packages. Every pure function is tested, including the parser, dollar formatter, dedup hash, Slack formatter, and Claude response parser.

---

## What I learned

**PDF parsing is complex.** Every library has encoding edge cases. Shell out to `pdftotext` rather than fighting a pure Go library that cannot read the font.

**Content-aware parsing beats positional parsing.** PDF formats shift. Detecting fields by appearance — date regex, dollar regex, or keyword prefix — is far more resilient than counting lines.

**Dedup first, enrich second.** Persisting the raw record before calling Claude ensures a failed enrichment does not lose data. Re-running only enriches missed items.

**Claude Haiku is excellent at structured extraction.** For a background pipeline, it costs almost nothing and produces actionable output. The system prompt context makes a real difference.

**Build the storage layer before the scraper.** The schema clarifies the data's shape, and everything upstream aligns with it.

---

## What's next

### Phase 16 — LLM Provider Abstraction

- Abstract LLM calls into a provider-agnostic interface
- Support for OpenAI, Groq, and local Ollama (for testing and cost-saving)
- Fallback logic: if Claude is down or out of credits, automatically retry with GPT-4o-mini

### Phase 17 — Airport Disruption Alert System

- A dedicated service (`alertd`) to monitor airports for flight cancellations and weather alerts
- Real-time Slack alerts for the hotel operations team
- Integration with local hotel inventory to include "rooms available"

### Phase 18 — Contact Enrichment

- Auto-surface decision-maker contacts (project managers, coordinators) using Hunter.io
- Attach contact details directly to the lead in Slack

---

## The honest update

The pipeline works and solves a real problem. My wife is cautiously optimistic, waiting for more leads in Slack. So am I.

The code is at [github.com/alvindcastro/groupscout](https://github.com/alvindcastro/groupscout). Next: wiring in the scheduler.
