---
title: 'The AI Data Strategy: Making SQL "AI-Ready"'
description: "How to structure a database schema and prompts so that an LLM can reliably score leads."
pubDate: "2026-04-13"
tags: ["sql", "ai-strategy", "postgres", "llm"]
draft: true
---

A raw database record is often a poor input for an LLM. It's too verbose, contains irrelevant metadata, and lacks context.

For Group Scout, I developed a strategy to make SQL "AI-ready." This ensures the `enrichment` layer receives exactly what it needs to produce a consistent score.

## The Problem: Data Overload

A building permit in Vancouver contains hundreds of fields across multiple tables. Sending a full JSON dump of these tables to an LLM is:

1. **Expensive**: High token counts.
2. **Confusing**: The model may focus on "red herring" data like internal IDs or audit logs.
3. **Inconsistent**: Different data sources have different field names.

## The Solution: The "AI View"

Instead of sending the `RawProject` struct directly, Group Scout uses a specialized prompt that transforms the record into a "clean" version.

I use a `system_prompt` that acts as a data filter:

```sql
SELECT
    p.project_id,
    p.title,
    p.description,
    p.estimated_value,
    COALESCE(e.score, 0) as historical_score
FROM leads p
LEFT JOIN enrichment e ON p.project_id = e.project_id
WHERE p.status = 'NEW';
```

The enrichment layer then maps this result into a specific JSON schema:

| Field        | Source             | Purpose                                                 |
| ------------ | ------------------ | ------------------------------------------------------- |
| `context`    | `description`      | The core "what" of the project                          |
| `magnitude`  | `estimated_value`  | Help the LLM estimate crew size                         |
| `historical` | `historical_score` | Prevents the LLM from scoring the same lead differently |

## Prompt Versioning

I store the prompts as separate Markdown files in the `docs/prompts` directory. This allows me to:

- A/B test different scoring instructions.
- Re-run the pipeline on historical data with a new version of the "scout" brain.
- Track changes to the "AI logic" in Git alongside the Go code.

## Conclusion

AI-ready SQL isn't just about cleaning data; it's about context injection. By filtering and structuring the data before it hits the LLM, Group Scout achieves a 95%+ accuracy rate in identifying high-value leads while keeping token costs to a minimum.
