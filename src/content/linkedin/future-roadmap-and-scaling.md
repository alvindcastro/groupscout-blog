---
title: "The Roadmap: Scaling from One City to Every Signal"
description: "How we're moving from a single source to a wider engine for regional leads."
pubDate: "2026-04-13"
draft: true
tags: ["roadmap", "scaling", "data-engineering", "groupscout"]
---

Building the first collector for Group Scout was a "proof of concept."

Now, it’s a template.

What started as a single source for one city is becoming a wider engine for regional leads.

The goal is to move from "one city" to "every signal":

- **More Municipalities**: Scaling the modular collector pattern to other cities in BC. Each one has its own PDF quirks, but the pipeline handles them.
- **Government Contracts**: Moving beyond building permits to government contract awards (via BC Bid)—a massive source for large, long-term crew hotel needs.
- **Similarity Search**: Using `pgvector` to identify and surface new leads that "look like" the ones that converted in the past.

The first Monday morning Slack message was the validator. Now, the challenge is building a pipeline that can handle any source, anywhere.

What’s next for Group Scout:
https://alvindcastro.github.io/groupscout-blog/posts/parsing-municipal-permits/
