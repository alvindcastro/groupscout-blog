---
title: "Film crews are just another signal"
description: "Construction crews were the starting point. Film and TV productions turned out to be an even stronger signal."
pubDate: '2026-04-11'
---

Construction crews were the first signal Group Scout was built around. They’re predictable, long‑running, and easy to reason about once you can see them early enough.

They’re not the only group that behaves that way.

It didn’t take long for my “client” — my wife — to point out another category hotel sales teams care about just as much: film and TV productions.

She kept coming back to the same theme: timing matters, and the best leads are the ones you see before everyone else does.

Large crews. Fixed schedules. Out‑of‑town specialists. And a real need for nearby accommodation for weeks or months at a time.

---

## Why film and TV matter

A feature film or TV series block can bring **80 to 300+ crew**, often flying in department heads and specialty roles who need to stay close to set.

From a hotel sales perspective, that looks a lot like construction:
- large groups
- defined timelines
- limited flexibility on location

In some cases, it’s an even better signal than permits. Productions mobilize quickly, and the booking window can be tight.

---

## Finding the data

Creative BC publishes an “In Production” list that solves that problem.

It’s public and regularly updated. It’s also not especially obvious.

Digging a little deeper turned up a Salesforce Visualforce endpoint that returns fully rendered HTML to a plain HTTP request.

Sometimes the best API is the one hiding in plain sight.

---

## Fitting into the pipeline

Each production maps cleanly into a `RawProject` and flows through the same steps as every other source:
- dedup
- LLM enrichment
- scoring
- Slack delivery

The only thing that changes is the enrichment prompt. Everything else stays the same.

That’s the point.

---

Permits tell you *what might get built*.  
Film and TV tell you *who is already mobilizing*.

Different signals. Same system.
