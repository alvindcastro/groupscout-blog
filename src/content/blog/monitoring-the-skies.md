---
title: "Monitoring the skies"
description: "From building permits to airport disruptions. Why watching the runway is a hotel signal."
pubDate: "2026-04-13"
draft: true
---

Group Scout started with building permits. It moved to film crews.

The third signal is the sky.

When a major airport faces disruption—winter storms, equipment failure, or massive delays—hundreds of travelers need a place to sleep. Immediately.

To a hotel near the airport, this is a surge in demand that arrives with zero warning.

---

## The Alertd Subsystem

I built `alertd` as a specialized subagent. It doesn't crawl permit databases; it watches for airport disruptions.

It monitors:

- Weather advisories for major hubs (YVR, YYC, YTO).
- Flight cancellation spikes.
- Public alerts from airport authorities.

---

## Why it matters

Building permits are a **long-term signal**. You have months to prepare.
Film crews are a **medium-term signal**. You have weeks.
Airport disruptions are a **real-time signal**. You have minutes.

A hotel sales manager can use this to:

- Adjust pricing in real-time.
- Open up "distressed inventory."
- Prepare the front desk for a surge of "walk-in" traffic.

---

## How it works

The `alertd` binary uses the same collector interface as the main pipeline.

When a disruption threshold is met:

1. It generates an alert.
2. The LLM produces a "copy-ready" outreach message for hotel staff.
3. A priority Slack notification is sent to the hotel's sales channel.

---

## The expansion

Group Scout isn't just a permit scraper. It's a signal aggregator.

Whether it's a shovel in the ground, a camera on a tripod, or a plane on the tarmac—it's all just data for the same pipeline.
