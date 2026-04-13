I started Group Scout with SQLite. It was simple, portable, and perfect—until it was not.

As the pipeline grew, I realized I needed more than just a place to store rows. I needed capability.

I migrated to PostgreSQL. Here is why:
1. Native UUIDs: Easier lead identification and data merging without collisions.
2. JSONB: I can store raw scraper output and query deep into it with binary indexing.
3. pgvector: By storing vector embeddings, I can perform similarity searches. This helps catch duplicates a simple hash misses.

I use `pgx` to keep my Go project CGO-free and `golang-migrate` for schema management.

SQLite handles hundreds of leads, but Postgres and `pgvector` prepare Group Scout for an intelligent future.

Full migration story: [Link]
