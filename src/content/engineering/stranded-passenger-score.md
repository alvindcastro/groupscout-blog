---
title: "The Stranded Passenger Score — Real-Time Signal Monitoring"
description: "How Group Scout uses a specialized subagent and a custom scoring formula to alert hotels to airport disruptions."
pubDate: "2026-04-14"
tags: ["go", "alertd", "real-time", "scoring", "monitoring"]
draft: true
---

Airport disruptions create sudden, unannounced demand for hotels near major hubs. Group Scout uses a dedicated subagent, `alertd`, to monitor these signals in real-time.

## The Stranded Passenger Score (SPS)

`alertd` does more than just report cancellations. It computes a **Stranded Passenger Score (SPS)** to filter out noise from minor delays. The formula weights multiple factors:

```go
SPS := (cancelled / scheduled) * avg_seats * connecting_pax_ratio * time_multiplier * duration_score
```

- **(cancelled/scheduled)**: The percentage of disrupted flights.
- **avg_seats**: Average aircraft capacity.
- **connecting_pax_ratio**: The likelihood of passengers requiring overnight lodging.
- **time_multiplier**: Nighttime cancellations trigger higher scores.
- **duration_score**: Prolonged events (e.g., fog or snow) receive higher weights.

A high SPS score indicates a genuine surge in lodging demand, alerting the hotel sales team to open distressed inventory or adjust pricing.

## Architecting a Subagent

`alertd` is a separate binary, distinct from the main lead pipeline. It runs on a different cadence and has independent failure modes.

The subagent implements the same `Collector` interface as the main system. It fetches data from weather advisories, airport cancellation feeds, and NOTAM (Notice to Air Missions) parsers.

## Alert Lifecycle

To prevent spam, `alertd` uses a state machine to manage notifications:

1.  **Watch**: A disruption begins but has not reached a threshold.
2.  **Alert**: The SPS exceeds the threshold for over thirty minutes.
3.  **Update**: The system updates the existing Slack message as the situation evolves.
4.  **Resolve**: The event ends, and the final impact is logged.

By updating a single message rather than posting new ones, `alertd` maintains a clean notification channel for hotel staff.

## Outreach Automation

When an alert fires, the system uses an LLM to generate an outreach draft for the front desk. This draft summarizes the situation (e.g., "Atmospheric river causing 40% cancellations at YVR") and suggests a response strategy.

Real-time signal monitoring transforms a hotel's approach from reactive to proactive, providing a critical advantage when every minute counts.
