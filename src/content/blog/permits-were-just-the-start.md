---
title: "Permits were just the start"
description: "Building permits were the obvious first signal, but they aren't the only — or best — one."
pubDate: '2026-04-09'
draft: true
---

Building permits were the first signal for Group Scout. They are public and updated regularly. Most importantly, they appear weeks before crews arrive. This gap between *permit issuance* and *crew mobilization* allows hotel sales teams to act rather than react.

Permits were a good start, but they cannot be the only signal.

---

## What makes a good signal

Not all public data is useful. Useful sources share several traits:
- public and updated predictably  
- visible *before* demand materializes  
- correlated with large, temporary groups  
- stable enough to parse reliably

Permits check most boxes, but not all demand appears there first.

---

## Why expand beyond permits

My wife asked a simple question:

> “Can this pick up film productions too?”

Film and TV shoots resemble construction crews: large teams, fixed timelines, and a need for nearby accommodation. Sometimes they are a stronger signal.

Expanding beyond permits became a requirement.

---

## One pipeline, many inputs

Every Group Scout source implements the `Collector` interface:
- collect raw data  
- map it to a `RawProject`  
- hand it to the dedup → enrich → notify pipeline  

Downstream processes remain the same. If a source produces a stable record and a dedup key, it works.

---

## Why this matters

Permits tell you *what might get built*.  
Other sources tell you *who is mobilizing*.

Different signals, same pipeline, same delivery. That's the Group Scout pattern.
