---
title: "GroupScout: Modern System Architecture for Hotel Sales"
description: "How we built a Go backend to automate hotel lead discovery."
pubDate: "2026-04-13"
draft: false
tags: ["system-architecture", "go", "backend", "software-engineering"]
---

Automating hotel lead generation requires more than a simple scraper.

We’ve built a robust architecture in **Go** to move from raw data to qualified leads at scale.

**The Layers:**

1. **Collector Layer:** Multi-source polling (Building Permits, RSS, Google News, etc.) via a common interface. Each source gets its own `Collector`.
2. **Storage Layer:** Clean Postgres storage. We use JSONB for flexible metadata and deduplicate leads at write-time using hashes.
3. **Enrichment Layer:** Rule-based scoring followed by a Claude API call for deep project analysis. We estimate crew sizes and project durations.
4. **Notification Layer:** Real-time Slack alerts for high-priority leads and a weekly Monday morning digest.

By keeping these layers separate, we can easily add new sources or swap LLMs without touching the core business logic.

Modular. Scalable. Fast.

#golang #systemdesign #backendengineering #softwarearchitecture #groupscout
