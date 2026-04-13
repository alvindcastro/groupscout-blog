---
title: "Build the foundation — store first, think later"
description: "Build the database layer before scraping code. The deduplication hash is the most important part of the pipeline."
pubDate: "2026-04-04"
draft: false
---

When I started Group Scout, I built the storage layer before the scraper. No data to store yet. No enrichment to persist. Just tables, schemas, and a dedup hash function.

It was the right call.

---

## The three tables

The database has three tables:

**`raw_projects`** is an append-only log. Every permit the pipeline collects goes here first. It stores the source, external ID, raw JSON data, timestamp, and a dedup hash. I never delete from this table.

**`leads`** is the enriched output. Each lead references a raw project and adds AI-generated intelligence: priority score, project type, estimated crew size, outreach timing, GC name, and notes. The sales team acts on this data.

**`outreach_log`** tracks who reached out to which lead, through which channel, and the outcome. I haven't wired it up yet, but the table exists. Building the schema early prepares the data model for the feature.

---

## The dedup hash — the most important thing in the pipeline

Every permit gets a SHA-256 hash derived from its folder number, address, and issue date. Before the pipeline spends a Claude API call, it checks if that hash exists.

```go
func HashProject(source, externalID, title string) string {
    h := sha256.New()
    h.Write([]byte(source + "|" + externalID + "|" + title))
    return hex.EncodeToString(h.Sum(nil))
}
```

If it exists, the pipeline skips it. If not, it enriches, persists, and notifies.

This matters for several reasons:

**The pipeline runs as often as needed.** One collector publishes permits weekly, but I run the pipeline twice a week to catch new reports. Without dedup, every second run would re-enrich permits and flood Slack with duplicates.

**A permit can appear in multiple reports.** Many municipal reports include permits from previous weeks. Without dedup, I would enrich the same permit multiple times.

**Extra runs cost nothing.** If the pipeline finds nothing new, the check takes milliseconds. No AI calls, no Slack messages. The run is effectively free.

---

## Store raw data before enriching

The pipeline writes the raw permit to `raw_projects` _before_ calling Claude.

API calls fail. Claude's API might be down, credits might be empty, or the JSON might not parse. If enrichment fails, I still have the raw permit data.

If enrichment failed, I wouldn't want to skip it forever. I use an `enriched_at` timestamp on the raw record. On startup, the pipeline finds raw records without a corresponding lead and retries them. Failed enrichments queue themselves automatically.

Designing for this from the start is easier than adding it later.

---

## Why foundation-first

If I built the scraper first and it worked, I might ship without a storage layer. A week later, I'd have untrustworthy data and be bolting a database onto a pipeline not designed for one.

Build the storage layer first. You know the data's shape. The scraper produces records that fit the schema. Enrichment fills the expected fields.

It's slower to start. The first week produces no visible output. But the pipeline lands on a designed foundation.

Next: scraping building permit PDFs from a city website with no API.
