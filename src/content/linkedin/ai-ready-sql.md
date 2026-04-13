---
title: "SQL is great. But it's not 'AI-Ready'."
description: "How I bridge the gap between Postgres and LLMs."
pubDate: "2026-04-13"
draft: false
tags: ["sql", "postgres", "llm", "data-strategy"]
---

A raw database record is often a mess. It's too verbose, full of internal IDs, and lacks the context an LLM needs to make a good decision.

For GroupScout, I had to figure out how to make SQL **"AI-Ready"**.

My strategy:

1. **The "AI View":** I don't send the full JSON dump. I use a specific SQL query that filters for only the "what" and "magnitude" of a project.
2. **Context Injection:** I map the raw data into a clean JSON schema the LLM can reliably score.
3. **Prompt Versioning:** Instructions for the "scout brain" live in Git, not in a random dashboard.

AI-ready SQL isn't just about cleaning data. It's about context.

By filtering and structuring the data _before_ it hits the LLM, I get better accuracy and lower token costs.

Don't just throw your DB at an LLM. Design the interface.

#sql #postgres #ai #datagovernance #golang
