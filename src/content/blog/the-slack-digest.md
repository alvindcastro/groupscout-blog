---
title: 'The Slack digest — no dashboard, no problem'
description: 'The Group Scout "UI" is a Slack message. meeting the sales team where they work minimizes friction.'
pubDate: '2026-04-12'
draft: false
---

Side projects eventually need an output. A web dashboard with filtering and CRM features would look impressive. But it would take weeks to build and might be ignored.

My wife needs to see leads on her phone Monday morning. She already uses Slack. Meeting the sales team where they work minimizes friction.

---

## How Slack incoming webhooks work

A Slack incoming webhook accepts a POST request with a JSON payload and posts a message to a channel. It requires no OAuth, just a URL from Slack settings.

The payload uses Slack's Block Kit for rich layouts. Sections, dividers, and buttons make it more expressive than plain text but simpler than a frontend.

---

## What a lead card looks like

Each lead gets a Block Kit card. The "View source document" link opens the original permit PDF for verification or context.

---

## Score emoji tiers

Claude's enrichment provides a priority score (1–10). The card header's emoji maps to a tier so the sales manager can scan and prioritize leads immediately.

---

## The smoketest flag

A `-testslack` flag on the smoketest command sends two fake leads to the webhook. This verifies the Block Kit layout without running the full pipeline — no real permits, Claude calls, or database entries.

```bash
go run ./cmd/smoketest/ -testslack
```

Two cards hit the Slack channel. If they look right, the formatter is working. If something's off — wrong emoji tier, missing field, broken link — I catch it before a real run.

This small improvement pays back immediately. When I change the card layout, the smoketest saves me from running the full pipeline to see the output.

---

## Why this was the right call

The Slack approach works because a sales manager makes calls; she doesn't check dashboards. The tool should remove friction.

A Monday morning Slack message puts leads where she already works. The pipeline reads PDFs, calls APIs, and sorts by priority. The sales manager decides who to call and how to pitch. This division of labor is the point.

Next: current status and future plans.
