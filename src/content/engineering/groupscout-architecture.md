---
title: "System Architecture Overview"
description: "How Group Scout is structured: collector, enrichment, storage, and notification layers."
pubDate: "2026-04-05"
tags: ["architecture", "go", "system-design"]
---

Group Scout is a Go backend structured in four layers. Each layer has one job.

## Collector

The collector layer polls public data sources — building permit databases, BC Bid RSS feeds, the Vancouver Convention Centre events page, Google News, and Eventbrite. Each source is a separate struct implementing a common interface. The pipeline runs on a schedule or via HTTP trigger (for n8n integration).

## Storage

All raw and enriched records land in Postgres. The schema is simple: one leads table with a JSONB column for source-specific metadata. Deduplication runs at write time using a hash of the source URL and project identifier.

## Enrichment

The enrichment layer takes a raw lead and produces a score. Two enrichers run in sequence: a rule-based pre-scorer (keyword matching, project type, dollar value) and a Claude API call that estimates crew size, project duration, and probability of out-of-town workers. The Claude call returns a priority score (1–10) and a plain-English rationale.

## Notification

Scored leads above a threshold trigger a Slack message immediately. A weekly digest runs every Monday morning and summarizes the top leads from the prior week. The `alertd` binary handles a separate use case: airport disruption monitoring that generates hotel-targeted outreach copy.
