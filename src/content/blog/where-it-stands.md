---
title: "Where it stands — and what's next"
description: 'The pipeline runs. Real permits are flowing. Here is the current state of blockscout and what comes next.'
pubDate: '2026-04-03'
heroImage: '../../assets/blog-placeholder-3.jpg'
---

The pipeline runs.

One command:

```bash
go run ./cmd/server/ --run-once
```

What happens:
1. Scrapes the Richmond weekly reports page for the latest PDF URL
2. Downloads the PDF, extracts text with `pdftotext`, parses 20 permit records
3. Filters: commercial sub-types with value above threshold
4. Checks each permit against the DB — skips anything already seen
5. Calls Claude to enrich new permits into scored leads
6. Posts a Block Kit digest to Slack

On the most recent run, 3 of 20 permits passed the filter:
- Hotel — 8640 Alexandra Road
- Restaurant — 5599 Cooney Rd
- Retail — 6553 Park Road

All three hit the enrichment stage. The Claude API key was out of credits at the time (a very relatable side project problem — it's since been topped up), but the pipeline code is fully wired and tested. When the credits were there, the enrichment worked correctly.

**Test coverage:** 32 unit tests across the collector, enrichment, and notify packages. Every pure function tested — the parser, the dollar formatter, the dedup hash, the Slack formatter, the Claude response parser.

---

## What I learned

**PDF parsing is never as simple as it looks.** Every library has encoding edge cases. When in doubt, shell out to `pdftotext` rather than fighting a pure Go library that can't read the font.

**Content-aware parsing beats positional parsing for messy real-world data.** The PDF format shifts slightly between reports. Detecting fields by what they look like — date regex, dollar regex, keyword prefix — is far more resilient than counting lines.

**Dedup first, enrich second.** Persisting the raw record before calling Claude means a failed enrichment doesn't lose the data. Re-running only re-enriches what was missed.

**Claude Haiku is remarkably good at structured extraction for ~$0.001 per call.** For a background pipeline running weekly on a handful of permits, it costs almost nothing and produces actionable output. The system prompt context — "you are a lead analyst for this specific hotel" — makes a real difference in output quality.

**Build the storage layer before the scraper.** Sounds backwards. Isn't. The schema clarifies what shape the data needs to be, and everything upstream falls into place around it.

---

## What's next

### Phase 3 — Harden and automate
- Schedule the pipeline to run Wednesday + Sunday at 8am automatically (an in-process cron scheduler, so it runs unattended)
- Add a rule-based pre-scorer that filters obviously low-quality permits before spending a Claude API call
- Add BC Bid contract parsing as a second data source — government contract awards are another strong signal for incoming crews

### Phase 4 — More signals
- Google News RSS for project announcements that appear before permits are even issued
- Instant Slack ping when a lead scores 9 or 10 (not just the weekly digest — immediate alert)
- Claude-drafted outreach email per lead, included in the digest so the sales manager can copy-paste and send

### Phase 5 — Production
- Docker container
- VPS deployment (Railway or a small Digital Ocean droplet)
- SQLite → PostgreSQL for production reliability
- Versioned migrations

---

## The honest update

The pipeline works. The problem it's solving is real. The person it's being built for is cautiously optimistic, which in hotel sales terms means she's waiting to see actual leads in Slack before she gets excited.

That's fair. So am I.

The code is at [github.com/alvindcastro/blockscout](https://github.com/alvindcastro/blockscout). This blog will keep tracking the build as it progresses. Next up: wiring in the scheduler so this stops being something I run manually and starts being something that just runs.
