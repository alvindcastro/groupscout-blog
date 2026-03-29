---
title: 'Building the foundation — store first, think later'
description: 'Why I built the database layer before writing a single line of scraping code, and why the deduplication hash is the most important thing in the whole pipeline.'
pubDate: '2026-03-30'
draft: true
---

When I started building groupscout, I did something that felt a little backwards at first: I built the storage layer before I built the scraper.

No data to store yet. No enrichment to persist. Just tables, schemas, and a dedup hash function — sitting there waiting.

It turned out to be the right call.

---

## The three tables

The database has three tables:

**`raw_projects`** is an append-only log. Every permit the pipeline collects goes here before anything else happens. Source, external ID, raw data as JSON, timestamp, and a dedup hash. Nothing is ever deleted from this table.

**`leads`** is the enriched output. Each lead references a raw project, and adds all the AI-generated intelligence: priority score, project type, estimated crew size, outreach timing, GC name, notes. This is what the sales team actually acts on.

**`outreach_log`** is for future use — tracking who reached out to which lead, through which channel, with what outcome. Not wired up yet, but the table is there. Building the schema early means the data model is ready when the feature is.

---

## The dedup hash — the most important thing in the pipeline

Every permit gets a SHA-256 hash derived from its folder number, address, and issue date. Before the pipeline spends a Claude API call enriching a permit, it checks whether that hash already exists in the database.

```go
func HashProject(source, externalID, title string) string {
    h := sha256.New()
    h.Write([]byte(source + "|" + externalID + "|" + title))
    return hex.EncodeToString(h.Sum(nil))
}
```

If it exists: skip. If it doesn't: enrich, persist, notify.

This matters for a few reasons.

**The pipeline can run as often as needed.** Richmond publishes permits weekly, but I run the pipeline on Wednesday and Sunday — twice a week — to make sure I catch new reports regardless of exactly when they're published. Without dedup, every second run would re-enrich the same permits and flood the Slack channel with duplicates.

**A permit can appear in multiple weekly reports.** Richmond's reports sometimes include permits from previous weeks. Without dedup, you'd enrich the same permit multiple times and generate duplicate leads.

**It costs nothing to run extra.** If the pipeline runs on a Sunday and finds nothing new, the dedup check takes milliseconds. No AI calls, no Slack messages. The run is effectively free.

---

## Store raw data before enriching

The other design choice I'm glad I made: the pipeline writes the raw permit to `raw_projects` *before* calling Claude, not after.

Here's why this matters: API calls fail. Claude's API might be down, my credit balance might be empty (it was, for a while), the JSON response might not parse correctly. If enrichment fails for any reason, I still have the raw permit data in the database.

The next time the pipeline runs, it checks the hash. The raw record already exists, so... it skips it. Wait — that's a problem. If enrichment failed, I'd never retry it.

The fix is a simple flag: `enriched_at` timestamp on the raw record. On startup, the pipeline looks for any raw records that have no corresponding lead and re-tries their enrichment. Failed enrichments don't disappear — they queue themselves for the next run automatically.

This is the kind of thing that's annoying to add later once you have real data in production. Much easier to design for it from the start.

---

## Why foundation-first

The logic I follow: if you build the scraper first and it works, you're excited and you ship without a proper storage layer. Then a week later you have data you can't trust because there's no dedup, and you're trying to bolt a database onto a pipeline that wasn't designed for one.

Build the storage layer first. You know exactly what shape the data needs to be in. The scraper becomes an exercise in producing records that fit the schema. The enrichment becomes an exercise in filling in the fields the schema expects.

It's slower to start. The first week of work produces no visible output — just a database that boots and applies its schema cleanly. But the pipeline, when it eventually runs, lands on a foundation that was designed to hold it.

Next: the part that was not clean at all — scraping building permit PDFs from a city website that has never heard of an API.
