### LLMs + pgvector — A pragmatic approach to AI-enriched leads

For Group Scout, the "AI" isn't the whole product. It’s an assistive layer on top of a deterministic pipeline.

Here’s the breakdown:

1. **LLM enrichment:** I use Claude Haiku to read building permits and infer what a human would—crew sizes, project duration, and "out-of-town" likelihood. It costs less than a tenth of a cent per lead.
2. **RAG foundations:** By moving to PostgreSQL + `pgvector`, I’ve started storing embeddings of each lead directly in the database.
3. **Similarity searches:** This unlocks the ability to do "Leads like this one" searches—matching new permits against a historical database of "won" business.

The goal? A system where the Go backend handles the boring parts—data parsing and deduplication—and the LLM adds the "context" that makes the data actionable.

It’s not about finding the biggest model. It’s about building a system where the model earns its keep.

My approach to building an AI-enriched storage layer:
https://alvindcastro.github.io/groupscout-blog/posts/the-stack/
