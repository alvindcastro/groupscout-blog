---
title: 'The Slack digest — no dashboard, no problem'
description: 'The entire "UI" of blockscout is a Slack message. Here is why that was the right call, and what it actually looks like.'
pubDate: '2026-04-02'
draft: true
heroImage: '../../assets/blog-placeholder-2.jpg'
---

At some point in every side project, you have to decide where the output goes.

A web dashboard was the obvious answer. Build a nice UI, add filtering, maybe a CRM-lite to track outreach. It would look impressive.

It would also take weeks to build, require a login, and probably be ignored after the first month. I've seen this pattern too many times.

The actual requirement was simple: my wife needs to see this week's leads, on her phone, on Monday morning. She already has Slack.

So the entire "UI" is a Slack webhook.

---

## How Slack incoming webhooks work

A Slack incoming webhook is a URL that accepts a POST request with a JSON payload and posts a message to a specific channel. That's it. No OAuth. No app setup beyond generating the webhook URL in Slack settings. No SDK required.

The payload is Slack's Block Kit format — a structured JSON format for rich message layouts. Sections, dividers, buttons, markdown text. It's more expressive than plain text and much simpler than building a frontend.

---

## What a lead card looks like

Each lead in the digest gets its own Block Kit card:

```
🔥 Warehouse — 12500 Vulcan Way                              Score: 9/10
📍 12500 Vulcan Way  |  💰 $1,200,000 CAD  |  🏢 GC: BuildRight Contracting
📞 Contractor: BuildRight Contracting (604)555-0199  |  Applicant: ABC Developments Ltd
🕐 Outreach: Reach out now — crews mobilizing in 4–6 weeks
📝 GC is BuildRight. Check LinkedIn for travel coordinator.
📄 View source document
```

The "View source document" link goes directly to the Richmond PDF the permit came from. If you want to verify the details or pull more context, one click gets you there.

---

## Score emoji tiers

The priority score comes from Claude's enrichment (1–10). The emoji in the card header maps to the tier:

| Score | Emoji | What it means |
|---|---|---|
| 9–10 | 🔥 | Call today |
| 7–8 | ⚡ | High priority this week |
| 5–6 | 👀 | Worth monitoring |
| < 5 | 📌 | Low priority, FYI |

The sales manager can scan the digest and immediately know which leads to act on first. No ranking to do manually. No interpretation needed.

---

## The smoketest flag

One thing I added early: a `-testslack` flag on the smoketest command that sends two fake leads to the webhook with known data. This lets me verify the Block Kit layout without running the full pipeline — no real permits, no real Claude calls, no real DB.

```bash
go run ./cmd/smoketest/ -testslack
```

Two cards hit the Slack channel. If they look right, the formatter is working. If something's off — wrong emoji tier, missing field, broken link — I catch it before a real run.

This is the kind of small quality-of-life thing that pays back immediately. The first time I changed the card layout and needed to check it, the smoketest saved me from running the full pipeline three times trying to see the output.

---

## Why this was the right call

Here is the honest reason the Slack approach works: the sales manager's job is not to log into a dashboard and check for leads. Her job is to make calls. The tool should remove friction from that process, not add a new surface to manage.

A Slack message Monday morning means the leads are already in the place she's already working. No extra step. No new app. Just: here are the projects, here's why they matter, here's when to reach out.

The pipeline does the boring part — reading PDFs, calling APIs, sorting by priority. The sales manager does the human part — deciding who to call, how to pitch, what to offer.

That division of labour is the whole point.

Next: where the pipeline stands right now and what's coming next.
