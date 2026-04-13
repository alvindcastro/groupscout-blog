---
title: "Modular Collectors: Scaling Dirty Data Parsing"
description: "How to handle messy municipal PDFs without a 'universal' parser."
pubDate: "2026-04-13"
draft: false
tags: ["automation", "go", "data-scraping", "system-design"]
---

PDFs are where data dies. They’re for printing, not parsing.

In GroupScout, we needed to scrape building permits from dozens of municipal websites. Each city has its own layout, its own data fields, and its own weird PDF software.

My first mistake? Trying to build a "universal" permit parser.

It’s a trap.

Instead, I built a **modular collector pattern**.

1. **Specific Collectors:** Each city gets its own `Collector` implementation to handle its quirks.
2. **Standard Output:** Each collector produces a clean, standardized `Permit` struct.
3. **Fuzzy Parsing:** The collector extracts raw text. We let Claude handle the "fuzzy" interpretation later.

This approach scales. When we add a new city, we don't break the existing ones. We just write a new collector, plug it in, and the leads start flowing.

Don't build for everything. Build to be modular.

#golang #automation #datacollection #systemarchitecture #groupscout
