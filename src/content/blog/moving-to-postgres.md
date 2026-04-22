---
title: "Migrating to Postgres and pgvector"
description: "Why I moved from SQLite to PostgreSQL and how pgvector enables intelligent lead matching."
pubDate: "2026-04-18"
draft: false
---

SQLite was the obvious choice for Group Scout at the start. It is simple, portable, and requires no configuration.

As the pipeline grew from one source to many—including film productions and contract awards—storage demands changed. I recently migrated to PostgreSQL.

---

## Why the move?

The move was not about performance; SQLite handles hundreds of leads. It was about **capability**.

### 1. Native UUIDs

I switched to UUIDs for lead identification. Postgres handles them natively, making it safer to merge data without collisions.

### 2. JSONB for Raw Data

Every scraper collects raw data before mapping it to the lead format. I store this output in a `JSONB` column. Unlike the text-based JSON in SQLite, `JSONB` is binary and indexed; it allows querying deep into output without extra parsing.

### 3. pgvector: The Real Reason

The migration added `pgvector`. Storing lead descriptions as vector embeddings enables **similarity searches**.

Why does this matter?

- **Finding duplicates:** Vector similarity catches duplicates that a simple hash-based check misses.
- **Retrieval-Augmented Generation (RAG):** I can retrieve similar historical leads and feed them to Claude as context.
- **Learning from Wins:** I can match new leads against historical successes to prioritize similar projects.

---

## The Migration Path

I use `pgx` as the driver. It is a pure Go implementation and maintains the "no CGO" rule.

I use `golang-migrate` for the schema. The app detects the driver and applies the correct migrations: Postgres for production and SQLite for local tests.

A small Go script moved the existing data into the new Postgres container.

---

## What's Next?

Postgres and `pgvector` prepare the system for LLM abstraction and the Airport Alert System.

The stack is more serious, but the goal remains: delivering the right lead to the right person at the right time.
