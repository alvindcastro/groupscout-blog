---
title: "The Engine and the Operator: Why I Separated Code from Workflow"
description: "How a Go backend and n8n work together to build a reliable and flexible data pipeline."
pubDate: "2026-04-13"
draft: true
tags: ["architecture", "go", "n8n", "workflow", "groupscout"]
---

When building a data pipeline like Group Scout, it's tempting to write code for _everything_. A custom scheduler, a UI for manual overrides, a Gmail integration...

But as a solo dev, that's a trap. Every line of code I write is a line I have to maintain.

My solution? A "Bridge" architecture:

1. **The Engine (Go):** This is the core. It handles the "heavy lifting"—scraping PDFs, calling the Claude API, and managing the PostgreSQL database. It’s built for reliability and performance.
2. **The Operator (n8n):** This is the "glue." It handles the scheduling, the manual overrides, and the messy external integrations (like RSS feeds or Slack commands).

By exposing a few key endpoints in the Go backend (like `/run` and `/n8n/webhook`), I can build complex workflows in n8n in minutes that would have taken days to code in Go.

The result: A backend that stays focused on being a reliable data engine, and a workflow layer that’s easy to change without a redeploy.

More on the "Bridge" architecture here:
https://alvindcastro.github.io/groupscout-blog/posts/the-n8n-bridge/
