---
title: "My wife sells hotel rooms. I write Go."
description: "How watching someone do a hard job well convinced me to build a tool to make it a little easier."
pubDate: "2026-03-28"
draft: false
---

My wife is good at her job.

She manages sales at a hotel near a major airport. She finds businesses that need rooms—construction companies, sports teams, film productions, and government contractors—and convinces them to book at her hotel.

I watched her work before I saw the problem. Leads exist: a $50 million pipeline upgrade three kilometers away requires beds for sixty workers for eight months. Such a booking transforms a hotel sales team. The challenge is finding it before competitors do.

The current workflow requires manual digging: Google Alerts, permit databases, trade publications, and word of mouth. It works but is slow and reactive. By the time a construction company seeks a hotel, it has already begun calling around.

---

## The idea

What if I could get there first?

If the city issues a building permit for a $40M project, that project begins in eight to twelve weeks. The contractor is about to hire crews who will need lodging. Currently, no hotel has called the contractor because no one knows yet.

Group Scout lives in that window.

The concept is simple: monitor public data for signals, use AI to identify those that matter, and present the best leads to the sales team before the window closes.

---

## What I'm building

[groupscout](https://github.com/alvindcastro/groupscout) is a Go backend that:

1. **Collects** raw project data from public sources—building permit databases, government contract awards, RSS feeds, and infrastructure project pages.
2. **Enriches** each record using the Claude API—estimates crew size, project duration, and the likelihood of out-of-town workers, then generates a priority score with a reason.
3. **Stores** the leads in a database.
4. **Notifies** the sales team via Slack—a weekly digest on Monday mornings and immediate alerts for high scores.

The notification says: "Call about these projects this week, and here is why."

---

## Why I write

I write for several reasons.

First, I want to document the journey: the decisions, the mistakes, and what I learn about hotel sales. Applying software to an overlooked domain interests me.

Second, this exercise teaches me: I am mastering Go, structuring a pipeline, and using AI for enrichment.

Third, if this works, it might help others in specialized industries who have data but lack connections.

I have finished two phases. The pipeline runs; building permits are collected and parsed. Claude enrichment is wired up, and Slack notifications are ready.

The next post covers Phase 1: manual validation. My wife taught me the first rule of hotel sales: do not build the tool before you understand the job.

She was right.
