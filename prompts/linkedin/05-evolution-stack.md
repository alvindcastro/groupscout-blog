### Post 5: The "Evolution" Post (Focus on the Stack & Scaling)
*Best for: Talking about the transition from a "weekend hack" to something more robust.*

Moving from SQLite to Postgres is a rite of passage for side projects.

Group Scout started as a single Go binary and a local SQLite file. It was perfect for the first few weeks of prototyping.

But as I started adding `pgvector` for similarity searches (to find "leads like this one") and wiring in `n8n` for workflow automation, the infrastructure had to grow up.

The current stack:
- **Go**: The engine (concurrent, type-safe, and perfect for pipelines).
- **Postgres + pgvector**: For production-grade storage and future RAG capabilities.
- **n8n**: The "bridge" that handles scheduling and manual overrides without cluttering the Go code.
- **Docker Compose**: To keep it all reproducible.

No web framework. No ORM. Just the standard library and a few battle-tested tools.

More on the stack choices behind the scenes:
https://alvindcastro.github.io/groupscout-blog/posts/the-stack/
