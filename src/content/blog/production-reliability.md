---
title: "Production reliability: Sentry, logging, and health checks"
description: "Transitioning from a local script to a production-grade system with error logging and health monitoring."
pubDate: "2026-04-14"
draft: true
---

A scheduled data pipeline is only as good as its visibility. It will fail: external sites change, PDFs corrupt, and API credits expire.

I focused Group Scout on system reliability.

I added these features for production readiness:

---

## 1. Structured Logging with `log/slog`

I replaced `fmt.Println` statements. They suffice for debugging but are difficult to query in production.

I moved to `log/slog`. Every log message is now JSON:

```json
{
  "time": "2026-04-12T14:30:05Z",
  "level": "INFO",
  "msg": "collected projects",
  "source": "city_permits",
  "count": 20
}
```

I pipe these logs into **Grafana Loki** and run queries, such as "How many permits did I collect last week?" or "Show all errors from the Claude service."

---

## 2. Sentry for Error Tracking

Structured logging provides visibility; Sentry provides **interruption**.

If a scraper fails or an API call returns a 500, I want immediate notice. Sentry captures these errors in real-time with stack traces and context.

Initializing Sentry at the start of `main()` provides coverage across the app. I know which line of code failed and on which URL.

---

## 3. Health Checks

A Docker container can run while the application inside is deadlocked or disconnected.

I added a `/health` endpoint that:

- **Pings the Database:** verifies the Postgres connection.
- **Checks Env Vars:** ensures critical configuration is present.
- **Validates Scraper State:** confirms scrapers are ready.

Docker and n8n use this endpoint to verify system health before triggering a run.

---

## 4. Pipeline Metrics

I log basic metrics for every run:

- Time to complete
- Projects collected vs. leads created
- Priority score distribution
- API cost estimation

This data reveals efficiency over time.

---

## Why bother?

Group Scout is a small project, but production readiness brings peace of mind. A self-monitoring system allows me to build new signals instead of debugging old ones.

Next: orchestrating collectors and the admin UI.
