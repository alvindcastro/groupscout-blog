---
title: 'My wife sells hotel rooms. I write Go.'
description: 'How watching someone do a hard job well convinced me to build a tool to make it a little easier.'
pubDate: '2026-03-28'
draft: false
---

My wife is good at her job.

She's a sales manager at a hotel near a major airport. She finds businesses that need many rooms and convinces them to book at her hotel. Construction companies, sports teams, film productions, and government contractors make up her world.

I watched her work before seeing the problem. Leads exist. A $50 million pipeline upgrade starts three kilometres from the hotel, and a rotating crew of 60 workers needs beds for eight months. That booking changes a hotel sales team's life. The problem: finding it before competitors do.

The current workflow requires manual digging: Google Alerts that surface news articles days late, permit databases that require specific knowledge, trade publications, and word of mouth. It works, but it's slow and reactive. By the time a construction company looks for hotels, they've already started calling around.

---

## The idea

What if I could get there first?

If a building permit for a $40M commercial project just issued, that project starts in 8–12 weeks. The GC is about to hire crews. Those crews need somewhere to stay. Right now, no hotel has called the GC because no one knows yet.

Group Scout lives in that window.

The concept is simple: monitor public data sources for signals, use AI to identify those that matter, and surface the best ones to the sales team before the window closes.

---

## What I'm building

[groupscout](https://github.com/alvindcastro/groupscout) is a Go backend that does a few things:

1. **Collects** raw project data from public sources — building permit databases, government contract awards, RSS feeds, and infrastructure project pages
2. **Enriches** each record using the Claude API — estimates crew size, project duration, and the likelihood of out-of-town workers, then generates a priority score with a plain-English reason
3. **Stores** the leads in a database
4. **Notifies** the sales team via Slack — a weekly digest on Monday mornings and an immediate alert for high scores

The notification says: "Call about these projects this week, and here's why."

---

## Why I'm writing about it

A couple reasons.

First, I want to document the journey — the decisions, the mistakes, and what I learn about hotel sales. Applying software to a domain most developers never think about interests me.

Second, this exercise teaches me. I'm mastering Go, learning to structure a real pipeline, and using AI enrichment usefully rather than just impressively.

Third, if this works, it might help someone else. Many specialized industries have the right data but lack the connections.

I've finished two phases. The pipeline runs. Real building permits are being collected and parsed. The Claude enrichment is wired up (though currently blocked on API credits). Slack notifications are formatted and ready.

The next post covers Phase 1: how I validated the idea manually before writing code. My wife taught me the first rule of hotel sales: don't build the tool before you understand the job.

She was right.
