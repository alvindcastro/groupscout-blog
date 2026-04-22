---
title: "The Multi-Driver Storage Strategy — Go, Postgres, and SQLite"
description: "How to support production-grade Postgres and local-first SQLite in a single Go codebase without code duplication."
pubDate: "2026-04-18"
tags: ["go", "sql", "postgres", "sqlite", "architecture"]
draft: true
---

Production systems require the power of PostgreSQL for native UUIDs, JSONB indexing, and vector support. Local development, however, benefits from the zero-configuration simplicity of SQLite. Group Scout supports both with a single set of interfaces.

## Dual-Driver Support

The system determines the driver at startup based on the `DATABASE_URL` prefix. It uses the `pgx` driver for Postgres and a Go-native driver for SQLite.

A single `Store` interface abstracts the operations, allowing the pipeline to remain agnostic of the underlying database.

```go
type LeadStore interface {
    Insert(ctx context.Context, lead Lead) error
    ListNew(ctx context.Context) ([]Lead, error)
    UpdateStatus(ctx context.Context, id string, status string) error
}
```

## SQL Rebinding

PostgreSQL uses `$1, $2` for placeholders; SQLite uses `?`. To avoid writing two sets of queries, Group Scout uses a `Rebind()` function that dynamically converts standard `?` placeholders into the driver-specific format at runtime.

```go
func (s *postgresStore) ListNew(ctx context.Context) ([]Lead, error) {
    query := s.Rebind("SELECT * FROM leads WHERE status = ?")
    return s.db.Query(ctx, query, "NEW")
}
```

## Migration Strategies

Managing schemas across two different SQL dialects requires care.

- **Postgres**: Uses `golang-migrate` for versioned, incremental schema updates.
- **SQLite**: Maintains an idempotent, inline schema that the application applies on startup if the database file is new.

## Vector Embeddings

Similarity search presents the biggest challenge. Group Scout uses `pgvector` on Postgres for production-grade vector indexing. On SQLite, it falls back to a custom Go-native implementation for development.

This strategy ensures that the application remains portable. A developer can clone the repository and run the full pipeline locally with SQLite, while production benefits from the stability and performance of a managed Postgres instance.
