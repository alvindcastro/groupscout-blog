---
title: 'My wife sells hotel rooms. I write Go.'
description: 'How watching someone do a hard job well convinced me to build a tool to make it a little easier.'
pubDate: '2026-03-28'
---

My wife is good at her job.

She's a sales manager at a hotel near YVR — which means her job is to find businesses that need to book large numbers of rooms and convince them to book at her hotel before they book anywhere else. Construction companies. Sports teams. Film productions. Government contractors. That's her world.

I watched her work for a while before I started to see the problem. It's not that the leads don't exist — they absolutely do. A $50 million pipeline upgrade starts up three kilometres from the hotel, and suddenly you've got a rotating crew of 60 workers who need somewhere to sleep for eight months. That's a genuinely life-changing booking for a hotel sales team. The problem is *finding it* before it finds your competitors.

The current workflow is a lot of manual digging. Google Alerts that surface news articles days after the fact. Permit databases that require you to know what you're looking for. Trade publications. Word of mouth. It works, but it's slow, and it's reactive. By the time a construction company is actively looking for hotels, they've already started calling around.

---

## The idea

What if you could get there first?

If a building permit for a $40M commercial project just got issued, that project is probably starting in 8–12 weeks. The GC is about to hire crews. Those crews are going to need somewhere to stay. And right now, nobody from any hotel has called the GC yet — because nobody knows yet.

That's the window. That's where Group Scout is supposed to live.

The concept is simple: monitor public data sources for signals, run them through AI to figure out which ones actually matter, and surface the best ones to the sales team before the window closes.

---

## What I'm building

[groupscout](https://github.com/alvindcastro/groupscout) is a Go backend that does a few things:

1. **Collects** raw project data from public sources — building permit databases, government contract awards, RSS feeds, infrastructure project pages
2. **Enriches** each record using the Claude API — estimates crew size, project duration, likelihood of out-of-town workers, and generates a priority score with a plain-English reason
3. **Stores** the leads in a database
4. **Notifies** the sales team via Slack — a weekly digest on Monday mornings, and an immediate alert for anything that scores really high

No dashboard. No login. Just a Slack message that says: "here are the projects you should be calling about this week, and here's why."

---

## Why I'm writing about it

A couple reasons.

First, I want to document the journey while it's happening — the decisions, the mistakes, what I learn about hotel sales that I definitely didn't expect to learn. There's something interesting about applying software to a domain that most developers never think about.

Second, this is genuinely a learning exercise for me. I'm getting more comfortable with Go, learning how to structure a real pipeline (not just toy code), and figuring out how to use AI enrichment in a way that's actually useful rather than just impressive-sounding.

Third — if this works, maybe it helps someone else in a similar situation. There are a lot of people in specialized industries where the right data exists but nobody's connected it up yet.

We're about two phases in. The pipeline runs. Real building permits are being collected and parsed. The Claude enrichment is wired up (though currently blocked on API credits — a very relatable problem). Slack notifications are formatted and ready.

The next post will be about Phase 1: how we validated the whole idea manually before writing a single line of code. Because the first thing my wife taught me about hotel sales is that you don't build the tool before you understand the job.

She was right.
