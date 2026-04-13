---
title: "The alertd Subsystem — Monitoring Real-Time Signals in Go"
description: "Architecting a specialized binary for real-time airport disruption alerts and outreach automation."
pubDate: "2026-04-14"
tags: ["go", "real-time", "monitoring", "alertd"]
draft: true
---

Group Scout began with building permits, which provide long-term signals for hotel demand. I built the `alertd` subsystem to monitor real-time signals, specifically airport disruptions.

## The Design

`alertd` is a specialized binary that uses the same `Collector` interface as the main pipeline. This design choice allowed me to reuse the existing infrastructure for fetching, storing, and notifying.

The subagent focuses on three signals:

1.  **Weather Advisories**: Monitoring major hub advisories.
2.  **Flight Cancellation Spikes**: Tracking surges in canceled arrivals.
3.  **Public Alerts**: Parsing RSS feeds and public notices from airport authorities.

## Real-Time Thresholds

Unlike the main pipeline, which runs on a schedule, `alertd` can trigger immediate notifications. It evaluates data against a threshold: when a disruption reaches a specific severity score, it fires a priority Slack message.

The system uses a Go ticker for polling:

```go
func (a *AlertDaemon) Run(ctx context.Context) error {
    ticker := time.NewTicker(15 * time.Minute)
    defer ticker.Stop()

    for {
        select {
        case <-ticker.C:
            if err := a.processSignals(ctx); err != nil {
                slog.Error("signal processing failed", "error", err)
            }
        case <-ctx.Done():
            return ctx.Err()
        }
    }
}
```

## Outreach Automation

When a disruption is detected, the `alertd` subsystem does more than just notify. It uses the LLM enrichment layer to generate a contextual outreach draft for the hotel staff.

The prompt for this generation is narrow. It informs the staff why the demand is surging (e.g., a winter storm at a major hub) and suggests an outreach strategy.

## Conclusion

By using the same abstractions for both long-term and real-time signals, I built a flexible monitoring system. The `alertd` subagent ensures that hotel sales teams have actionable data when every minute matters.
