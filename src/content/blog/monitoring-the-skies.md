---
title: "Monitoring the skies"
description: "From building permits to airport disruptions. Why watching the runway is a hotel signal."
pubDate: "2026-04-13"
draft: true
---

Group Scout began with building permits and moved to film crews.

The third signal is the sky.

When a major airport faces disruption—winter storms, equipment failure, or massive delays—hundreds of travelers immediately need lodging.

To a hotel near the airport, this surge in demand arrives without warning.

---

## The Alertd Subsystem

I built `alertd` as a specialized subagent. It watches for airport disruptions rather than crawling permit databases.

It monitors:

- Weather advisories for major hubs
- Flight cancellation spikes
- Public alerts from airport authorities

---

## Why it matters

Building permits are long-term signals; you have months to prepare. Film crews are medium-term; you have weeks. Airport disruptions are real-time; you have minutes.

A sales manager uses this to:

- Adjust pricing
- Open "distressed inventory"
- Prepare the front desk for walk-in traffic

---

## How it works

The `alertd` binary uses the same collector interface as the main pipeline.

When a disruption reaches a threshold:

1. It generates an alert.
2. The LLM produces an outreach message for staff.
3. The system sends a priority Slack notification to the sales channel.

---

## The expansion

Group Scout is a signal aggregator, not merely a permit scraper.

A shovel in the ground, a camera on a tripod, or a plane on the tarmac—all provide data for the same pipeline.
