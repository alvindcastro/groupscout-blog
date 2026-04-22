---
title: "AI-Ready SQL — Database Views as LLM Prompts"
description: "How to use Postgres views to transform raw database records into dense, context-rich strings for LLM enrichment."
pubDate: "2026-04-19"
tags: ["sql", "ai", "postgres", "llm", "prompts"]
draft: true
---

Early versions of Group Scout built LLM prompts using Go's `fmt.Sprintf`. This method is fragile and scatters context logic across the codebase. I replaced it with a single Postgres view that pre-builds a dense, LLM-ready context string.

## The Context Problem

A building permit or government bid contains hundreds of fields. Sending a raw JSON dump to an LLM is expensive and confusing. The model might focus on irrelevant metadata or fail to connect related fields.

The enrichment layer requires a clean, structured summary of the lead.

## The v_lead_context View

I created a `v_lead_context` view in Postgres. It joins `leads` and `raw_projects` to produce a single `context_text` column. This column concatenates key fields into a compact format optimized for token efficiency.

```sql
CREATE VIEW v_lead_context AS
SELECT
    l.id,
    l.source,
    'Source: ' || l.source ||
    '. Title: ' || l.title ||
    '. Location: ' || COALESCE(l.location, 'unknown') ||
    '. Value: ' || COALESCE(l.project_value::text, 'unknown') ||
    '. Type: ' || COALESCE(l.project_type, 'unknown') ||
    '. Score: ' || l.priority_score || '/10.' AS context_text
FROM leads l
LEFT JOIN raw_projects rp ON l.raw_project_id = rp.id;
```

## Why This Wins

By moving prompt assembly into the database, I gained three technical advantages:

1.  **Centralization**: The prompt structure lives in a migration file, not hidden in Go code.
2.  **RAG Hooks**: The `context_text` column also serves as the input for vector embeddings. The same string used for enrichment is also used for similarity search.
3.  **Performance**: The enrichment layer simply queries `SELECT context_text FROM v_lead_context WHERE id = $1`. The database handles the string concatenation and joining, keeping the Go application stateless.

## Handling Dialects

SQLite lacks the concatenation operators and specific join behavior of Postgres. To maintain compatibility, the `LeadStore` interface includes a `GetContext()` method.

When running on Postgres, it queries the view. When on SQLite, it falls back to a basic Go-based string builder. This ensures that the core AI-ready strategy benefits production without breaking local development.
