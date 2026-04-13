---
title: "System Architecture Overview"
description: "How Group Scout is structured: collector, enrichment, storage, and notification layers."
pubDate: "2026-04-05"
tags: ["architecture", "go", "system-design"]
draft: false
---

Group Scout is a Go backend with four layers; each has one job.

## Collector

The collector layer polls public data: building permit databases, RSS feeds, event pages, Google News, and Eventbrite. Each source implements a common interface. The pipeline runs on a schedule or via an HTTP trigger.

## Storage

All records land in Postgres. The schema is simple: one leads table with a `JSONB` column for metadata. Deduplication occurs at write time using a hash of the source URL and project identifier.

## Enrichment

The enrichment layer scores raw leads. Two enrichers run in sequence: a rule-based pre-scorer and a Claude API call. Claude estimates crew size, duration, and probability of out-of-town workers, then returns a priority score and rationale.

## Notification

Leads above a threshold trigger a Slack message. A weekly digest summarizes the top leads every Monday. The `alertd` binary monitors airport disruptions and generates outreach copy.
