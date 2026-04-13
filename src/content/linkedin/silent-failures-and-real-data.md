---
title: "The Silent Failures of Clean Data"
description: "Why real-world messy data is the only thing that actually tests your system."
pubDate: "2026-04-13"
draft: true
tags: ["debugging", "data-quality", "go", "groupscout"]
---

> The hardest bugs to find are the ones that don’t error out.

When I was building the building permit parser for Group Scout, I hit a wall. The collector was running perfectly. Zero errors. Zero panics.

But it was returning zero leads.

It turned out to be a "silent failure" on two fronts:

1. **The Library Silence**: A pure Go PDF library I was using couldn't handle non-standard font encoding. It didn't complain; it just returned an empty string.
2. **The "Off-by-One" Logic**: After switching to `pdftotext`, I realized my parser was assigning the "Permit Count" (usually `1`) to the "Construction Value" field. So every $1M project looked like it was worth $1.

The lesson? Don't just test with "clean" fixture data. Use the messiest, real-world output you can find.

If I hadn't dumped the raw text and looked at it line-by-line, I’d still be wondering why the system thought every construction project in the city cost exactly one dollar.

More on the bugs that looked like config issues:
https://alvindcastro.github.io/groupscout-blog/posts/cracking-the-pdf/
