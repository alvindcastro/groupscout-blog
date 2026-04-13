---
title: "Migrating to Postgres and pgvector"
description: "Why I moved from SQLite to PostgreSQL and how pgvector enables intelligent lead matching."
pubDate: '2026-04-13'
---

SQLite was the obvious choice for Group Scout's start. It is simple, portable, and zero-configuration.

But as the pipeline grew from one source to many — including film productions and contract awards — the storage demands changed. I recently completed the migration to PostgreSQL.

---

## Why the move?

The move wasn't about performance; SQLite handles hundreds of leads easily. It was about **capability**.

### 1. Native UUIDs
I switched to UUIDs for lead identification. Postgres handles this natively with `gen_random_uuid()`, making it safer to merge data without ID collisions.

### 2. JSONB for Raw Data
Every scraper collects "raw" data before mapping it to my lead format. I store this output in a `JSONB` column. Unlike SQLite's text-based JSON, `JSONB` in Postgres is binary and indexed. It allows me to query deep into the scraper output without extra parsing.

### 3. pgvector: The Real Reason
The migration's centerpiece was adding `pgvector`. Storing lead descriptions as vector embeddings enables **similarity searches**.

Why does this matter?
- **Finding duplicates:** Vector similarity catches duplicates where a simple hash-based check fails.
- **RAG (Retrieval-Augmented Generation):** I can retrieve "leads like this one" from the historical database and feed them into the Claude prompt as context. 
- **Learning from "Wins":** I can match new leads against historical "Won" business to boost the priority of projects that resemble successful past closes.

---

## The Migration Path

I used `github.com/jackc/pgx/v5` as the driver. It is a pure Go implementation, maintaining the "no CGO" rule.

I used `golang-migrate` for the schema. The app detects the database driver from the connection string and applies the correct migrations — Postgres for production and SQLite for local tests.

A small Go script (`scripts/migrate_to_postgres/main.go`) moved the existing data from the `.db` file into the new Postgres container.

---

## What's Next?

Postgres and pgvector prepare me for Phase 16: LLM abstraction and Phase 17: the Airport Alert System.

The stack is more serious, but the goal remains: getting the right lead to the right person at the right time.
