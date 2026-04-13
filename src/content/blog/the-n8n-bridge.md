---
title: "The n8n bridge — connecting code to workflow"
description: "Using n8n to handle the 'human' parts of the pipeline keeps the Go backend focused on its core."
pubDate: "2026-04-06"
draft: true
---

I avoid writing code for everything. Developers often want to write custom schedulers, web UIs, or Gmail integrations, but my time is limited and those features are not core.

The core of Group Scout is signal detection and enrichment. The workflow—scheduling, manual overrides, and CRM integration—is where n8n helps.

---

## What is n8n?

n8n is an open-source automation tool; it connects Group Scout to the world.

Instead of a complex scheduling system, I have a `/run` endpoint. n8n calls it every Monday at 9:00 AM.

Instead of a UI for adding leads from news articles, I use an n8n webhook. If my wife sees a project on LinkedIn, she can share the link to a Slack command; n8n then handles extraction and pushes it to Group Scout.

---

## The "Bridge" Architecture

The architecture looks like this:

1. **Group Scout (Go)**: the engine. It scrapes permit sites, communicates with Claude, and stores leads in PostgreSQL.
2. **n8n**: the operator. It determines when the engine runs and acts as a gateway for external data.

I have exposed key endpoints to n8n:

- `POST /run`: triggers collection and enrichment.
- `POST /n8n/webhook`: accepts JSON from n8n to create a lead from an external source.
- `POST /digest`: triggers the weekly summary.

---

## Why this makes sense for a solo dev

Iteration is fast.

Last week, I pulled leads from an RSS feed. Doing this in Go would require a new `Collector`, RSS parsing, rate limiting, and a redeploy.

In n8n, I connected an 'RSS Read' node and a 'Filter' node to the Group Scout webhook; it took ten minutes.

This keeps the Go codebase clean. The backend remains a reliable data engine, while external integrations live in a workflow tool for easy debugging.

---

## Security

Because these endpoints trigger costly AI calls, I secured them. I added Bearer token authentication in Go and set up 'Header Auth' in n8n. It is simple and effective.

Next: the municipal building permit collector.
