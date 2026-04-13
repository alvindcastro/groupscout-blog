---
title: "Permits were just the start"
description: "Building permits were the obvious first signal, but they aren't the only — or best — one."
pubDate: "2026-04-12"
draft: false
---

Building permits were the first signal for Group Scout. They are public and regular. Most importantly, they appear weeks before crews arrive; this gap between issuance and mobilization allows hotel sales teams to act.

Permits were a good start, but they are only one of many signals.

---

## What makes a good signal

Not all public data is useful. Useful sources share several traits:

- public and regular
- visible before demand materializes
- correlated with temporary groups
- stable and reliable

Permits check most boxes, but not all demand appears there first.

---

## Why expand beyond permits

My wife asked a simple question:

> “Can this pick up film productions too?”

Film and TV shoots resemble construction crews: they have large teams, fixed timelines, and a need for accommodation. Sometimes they provide a stronger signal.

Expanding beyond permits became a requirement.

---

## One pipeline, many inputs

Every Group Scout source implements the `Collector` interface:

- collect raw data
- map it to a `RawProject`
- hand it to the dedup → enrich → notify pipeline

Downstream processes remain the same. If a source produces a stable record and a deduplication key, it works.

---

## Why this matters

Permits tell you what might be built; other sources tell you who is mobilizing.

Different signals, same pipeline—that is the Group Scout pattern.
