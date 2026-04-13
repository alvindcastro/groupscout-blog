---
title: "Production reliability: Sentry, logging, and health checks"
description: "Transitioning from a local script to a production-grade system with error logging and health monitoring."
pubDate: '2026-04-14'
draft: true
---

A scheduled data pipeline is only as good as your visibility into its failures. It *will* fail: external sites change, PDFs corrupt, and API credits expire.

I shifted Group Scout's focus to "system reliability."

Here’s what I added to make it production-ready.

---

## 1. Structured Logging with `log/slog`

I replaced standard `fmt.Println` statements. They work for debugging but are hard to query in production.

I moved to `log/slog` (added in Go 1.21). Every log message is now JSON:

```json
{"time":"2026-04-12T14:30:05Z","level":"INFO","msg":"collected projects","source":"city_permits","count":20}
```

I can pipe these logs into **Grafana Loki** and run queries: *"How many permits did I collect last week?"* or *"Show all errors from the Claude enrichment service."*

---

## 2. Sentry for Error Tracking

Structured logging provides visibility; Sentry provides **interruption**.

If a scraper fails or an API call returns a 500, I want to know immediately. Sentry captures these errors in real-time with full stack traces and context.

Initializing Sentry at the start of `main()` gives coverage across the app. I know exactly which line of code failed on which PDF URL.

---

## 3. Health Checks

A Docker container can "run" while the app inside is deadlocked or disconnected.

I added a `/health` endpoint that:
- **Pings the Database:** Verifies the Postgres connection.
- **Checks Env Vars:** Ensures critical configuration (like API keys) is present.
- **Validates Scraper State:** Confirms scrapers are registered and ready.

Docker and n8n use this endpoint to verify the system is healthy before triggering a run.

---

## 4. Pipeline Metrics

I log basic metrics for every run:
- Time to complete
- Projects collected vs. leads created
- Priority score distribution
- API cost estimation

This data provides a clear picture of efficiency over time.

---

## Why bother?

Group Scout is a small project, but "production-ready" means **peace of mind**. A self-monitoring system lets me build new signals instead of debugging old ones.

Next: orchestrating collectors and building the admin UI.
