---
title: "The Slack digest — no dashboard, no problem"
description: 'The Group Scout "UI" is a Slack message. meeting the sales team where they work minimizes friction.'
pubDate: "2026-04-12"
draft: false
---

Side projects eventually need an output. A web dashboard with filtering and CRM features would be impressive, but it would take weeks to build and might be ignored.

My wife must see leads on her phone on Monday mornings. She already uses Slack; meeting the sales team where they work minimizes friction.

---

## How Slack incoming webhooks work

A Slack incoming webhook accepts a POST request with a JSON payload and posts a message to a channel. It requires only a URL from the Slack settings.

The payload uses Slack's Block Kit for rich layouts. Sections, dividers, and buttons are more expressive than plain text but simpler than a frontend.

---

## What a lead card looks like

Each lead receives a Block Kit card. A link opens the original permit PDF for verification.

---

## Score emoji tiers

Claude's enrichment provides a priority score from one to ten. The card's emoji maps to a tier, allowing the sales manager to prioritize leads immediately.

---

## The smoketest flag

A `-testslack` flag on the smoketest command sends two fake leads to the webhook. This verifies the Block Kit layout without the full pipeline.

```bash
go run ./cmd/smoketest/ -testslack
```

Two cards hit the Slack channel. If they look correct, the formatter is working; if not, I catch errors before a real run.

This improvement pays back immediately. When I change the layout, the smoketest saves me from running the full pipeline.

---

## Why this was the right call

The Slack approach works because a sales manager makes calls rather than checking dashboards. The tool should remove friction.

A Monday morning Slack message puts leads where she works. The pipeline reads PDFs, calls APIs, and sorts by priority; the sales manager decides whom to call and how to pitch. This division of labor is the point.

Next: current status and future plans.
