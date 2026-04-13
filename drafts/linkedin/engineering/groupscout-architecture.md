The architecture for Group Scout is built for simplicity and growth.

I used a modular system in Go:
1. Collectors: One implementation per data source (PDF, Salesforce, RSS). Each collector returns a list of raw projects.
2. Deduplicator: A SHA-256 hash or vector similarity search ensures we only process new data.
3. Enricher: An interface for LLMs (Claude) to add context like crew size and priority.
4. Notifier: A push mechanism for delivery to Slack via n8n.

The key to this design is the 'Job' orchestrator. It manages the flow—fetching, deduplicating, and enriching—without knowing the source of the data.

Group Scout is now an engine. To add a new signal, I only need to implement a new `Collector`.

Full architectural breakdown: [Link]
