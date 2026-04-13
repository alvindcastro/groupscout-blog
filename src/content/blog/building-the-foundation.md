---
title: "Build the foundation — store first, think later"
description: "Build the database layer before scraping code. The deduplication hash is the most important part of the pipeline."
pubDate: "2026-04-04"
draft: false
---

I built the storage layer before the scraper. I had no data to store and no enrichment to persist—only tables, schemas, and a deduplication hash function.

It was the right call.

---

## The three tables

The database contains three tables:

**`raw_projects`** is an append-only log. Every permit goes here first. It stores the source, external ID, raw JSON, timestamp, and a hash. I never delete from this table.

**`leads`** contains the enriched output. Each lead references a raw project and adds AI-generated data: priority, type, crew size, timing, contractor, and notes. The sales team acts on this information.

**`outreach_log`** tracks contacts, channels, and outcomes. Although not yet connected, the table exists. Building the schema early prepares the data model.

---

## The dedup hash — the pipeline's core

Every permit receives a SHA-256 hash derived from its folder number, address, and issue date. Before calling the Claude API, the pipeline checks if the hash exists.

```go
func HashProject(source, externalID, title string) string {
    h := sha256.New()
    h.Write([]byte(source + "|" + externalID + "|" + title))
    return hex.EncodeToString(h.Sum(nil))
}
```

If the hash exists, the pipeline skips the permit; otherwise, it enriches, persists, and notifies.

This approach succeeds for several reasons:

**The pipeline runs as needed.** Although one collector publishes permits weekly, I run the pipeline twice a week to catch new reports. Without deduplication, the second run would re-enrich permits and flood Slack with duplicates.

**A permit can appear in multiple reports.** Many municipal reports include permits from previous weeks. Deduplication prevents redundant enrichment.

**Extra runs cost nothing.** If the pipeline finds nothing new, the check takes milliseconds. With no AI calls or Slack messages, the run is free.

---

## Store raw data before enriching

The pipeline writes the raw permit to `raw_projects` _before_ calling Claude.

API calls fail. Claude might be down, credits might be exhausted, or the JSON might not parse. If enrichment fails, the raw data remains.

If enrichment fails, the pipeline should retry. I use an `enriched_at` timestamp on the raw record. On startup, the pipeline identifies raw records without a lead and retries them. Failed enrichments queue themselves.

Designing for this early is easier than adding it later.

---

## Why foundation-first

If I built the scraper first, I might ship without a storage layer. A week later, I would have untrustworthy data and a database bolted onto a mismatched pipeline.

Build the storage layer first. You then know the shape of the data. The scraper produces records that fit the schema, and enrichment fills the expected fields.

Starting is slower; the first week produces no visible output. But the pipeline rests on a designed foundation.

Next: scraping permit PDFs.
