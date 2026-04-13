---
title: 'The AI Data Strategy: Making SQL "AI-Ready"'
description: "How to structure a database schema and prompts so that an LLM can reliably score leads."
pubDate: "2026-04-13"
tags: ["sql", "ai-strategy", "postgres", "llm"]
draft: true
---

A raw database record is often poor input for an LLM. It is verbose, contains irrelevant metadata, and lacks context.

I developed a strategy to make SQL "AI-ready." This ensures the enrichment layer receives exactly what it needs to produce a consistent score.

## The Problem: Data Overload

A building permit in Vancouver contains hundreds of fields. Sending a full JSON dump to an LLM is:

- **Expensive**: high token counts.
- **Confusing**: the model may focus on irrelevant data, such as internal IDs.
- **Inconsistent**: different sources use different field names.

## The Solution: The "AI View"

Instead of sending the `RawProject` struct directly, Group Scout uses a specialized prompt to transform the record.

A `system_prompt` acts as a data filter:

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

I store prompts as Markdown files in `docs/prompts`. This allows me to:

- A/B test different scoring instructions.
- Re-run the pipeline on historical data.
- Track changes in Git.

## Conclusion

AI-ready SQL involves context injection, not just cleaning data. By filtering and structuring data before it reaches the LLM, Group Scout identifies high-value leads accurately while minimizing costs.
