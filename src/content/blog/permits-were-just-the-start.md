---
title: "Permits were just the start"
description: "Building permits were the obvious first signal. They turned out not to be the only — or even the best — one."
pubDate: '2026-04-09'
---

When I started Group Scout, building permits were the obvious first signal.

They’re public, updated regularly, and — most importantly — they show up weeks before crews actually arrive. That gap between *permit issued* and *crews mobilizing* is where hotel sales teams can get ahead instead of reacting.

Permits were a good place to start. They still are.

But once the pipeline was running end‑to‑end, it became clear they couldn’t be the only signal.

---

## What makes a good signal

Not all public data is useful.

The sources that actually matter tend to share a few traits:
- public and updated on a predictable cadence  
- visible *before* demand materializes  
- correlated with large, temporary groups  
- stable enough to parse without constant breakage  

Permits check most of these boxes — but not all demand shows up there first.

---

## Why expand beyond permits

This became obvious the first time my “client” — my wife — asked a simple question:

> “Can this pick up film productions too?”

For hotel sales, film and TV shoots behave a lot like construction crews: large teams, fixed timelines, and a real need for nearby accommodation. In some cases, they’re an even stronger signal.

Once that question was on the table, expanding beyond permits stopped being hypothetical and became a requirement.

---

## One pipeline, many inputs

Every source in Group Scout implements the same `Collector` interface:
- collect raw data  
- map it into a `RawProject`  
- hand it to the same dedup → enrich → notify pipeline  

Everything downstream stays the same.

If a source can produce a stable record and a dedup key, it works.

---

## Why this matters

Permits tell you *what might get built*.  
Other sources tell you *who is already mobilizing*.

Different signals. Same pipeline. Same delivery.

That’s the pattern Group Scout is built around.
