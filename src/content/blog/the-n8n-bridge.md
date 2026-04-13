---
title: "The n8n bridge — connecting code to workflow"
description: "Using n8n to handle the 'human' parts of the pipeline keeps the Go backend focused on its core."
pubDate: "2026-04-06"
draft: true
---

I avoid writing code for _everything_. Developers often want to write a custom scheduler, a web UI, or a Gmail integration from scratch. But my time is limited, and those features aren't the "core."

Group Scout's core is **signal detection and enrichment**. The "workflow" — when to run, manual overrides, and pushing data to a CRM — is where n8n helps.

---

## What is n8n?

n8n is an open-source workflow automation tool. It's the "glue" connecting Group Scout to the world.

Instead of a complex Go scheduling system, I have a `/run` endpoint. n8n calls it every Monday at 9:00 AM.

Instead of a UI for adding leads from news articles, I use an n8n webhook. If my wife sees a project on LinkedIn, she can (eventually) share the link to a Slack command. n8n handles extraction and pushes it to Group Scout.

---

## The "Bridge" Architecture

The architecture looks like this:

1.  **Group Scout (Go)**: The "engine." It scrapes permit sites, talks to Claude, and stores leads in PostgreSQL.
2.  **n8n**: The "operator." It decides _when_ the engine runs and acts as a gateway for external data.

I've exposed a few key endpoints to n8n:

- `POST /run`: Triggers the full collection and enrichment pipeline.
- `POST /n8n/webhook`: Accepts structured JSON from n8n to create a lead from an external source (like an RSS feed or a Google Sheet).
- `POST /digest`: Triggers the weekly summary notification.

---

## Why this makes sense for a solo dev

Iteration is fast.

Last week, I pulled leads from a construction news RSS feed. Doing this in Go would require a new `Collector`, RSS parsing, rate limiting, and a redeploy.

In n8n, I dragged an "RSS Read" node, added a "Filter" node, and connected it to the Group Scout `/n8n/webhook` node. It took 10 minutes.

This keeps the Go codebase clean. The backend is a reliable data engine. External integrations live in a visual workflow tool for easy debugging and change.

---

## Security (The "Bearer" minimum)

These endpoints trigger costly AI calls, so I secured them. I added Bearer token authentication middleware in Go. In n8n, I set up "Header Auth" with that token. It’s simple and effective.

Next: the municipal building permit collector.
