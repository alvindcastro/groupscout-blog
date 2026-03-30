---
title: "The problem — why you can't just wait for the phone to ring"
description: "Construction crew lodging is one of the most valuable segments in hotel sales. It's also one of the hardest to find. Here's why."
pubDate: '2026-03-28'
---

Let me explain the business case before I explain the code.

When a major construction project breaks ground near a hotel — a warehouse, a commercial development, a pipeline upgrade — the general contractor needs to house their crew. Not one or two people. We're talking 20 to 100+ workers, rotating in and out, for anywhere from three months to over a year.

That's extended-stay rates. Direct company billing. Weekly room blocks. Per-diem meal deals with the on-site restaurant. It is, without exaggeration, exactly the kind of business a hotel near an industrial area should be chasing.

The hotel sits minutes from YVR, has a large parking lot that fits work trucks, an on-site restaurant for meal plans, and early breakfast service. For a rotating construction crew, it checks every box.

So why doesn't this business just... show up?

---

## The timing problem

Here's the thing about construction crews: by the time they're actively looking for hotels, it's too late. They've already started calling around. The GC's travel coordinator has a shortlist. Someone already got there first.

The window to win this business isn't when they're booking. It's weeks or months earlier — when the project is being permitted, when the contract is being awarded, when the announcement hits the news. That's when you can call and say: "I saw your project at 12500 Vulcan Way just got permitted. We're three minutes away. Let me tell you about our extended-stay rates."

That call, made early, is almost always welcome. It solves a problem the GC was about to have. It positions the hotel as proactive rather than reactive. And it usually wins the business.

The problem is finding the signal early enough.

---

## The data is public — but not especially accessible

Here’s what surprised me when I started digging into this: the information is public.

The City of Richmond publishes weekly building permit reports. Every commercial permit — warehouse, office, restaurant, hotel, industrial — shows up there. Permit number, address, applicant, contractor, construction value, work type, issue date. All of it posted as a PDF every week.

I spent some time looking for an API or structured feed and didn’t find anything obvious. As far as I can tell, the primary way this information is published is still a weekly PDF on a city website. But regardless of the delivery mechanism, the data itself is there.

Some people may skim these reports, and some teams may catch pieces of them informally. But in practice, this isn’t something most hotel sales teams are tracking consistently or turning into a repeatable workflow. Google Alerts surface news articles days after the fact. Trade publications focus on the biggest projects and miss the mid‑sized ones that often make the best leads. Word of mouth helps, but it’s hit‑or‑miss.

So the situation is this: the signal exists, it’s public, it’s updated weekly — and the gap between “permit issued” and “crews mobilizing” is typically 8–12 weeks. Plenty of time to get a call in, if you know where to look.

---

## What Group Scout does with that

The idea behind Group Scout is simple: read the permits, filter for the ones that matter, figure out which ones are worth a phone call, and tell the sales team.

In practice that means:
- Scraping Richmond's weekly permit PDF
- Filtering for commercial sub-types above a dollar threshold (no interior renos, no residential)
- Sending each permit to an AI model that estimates crew size, project duration, likelihood of out-of-town workers, and a priority score
- Posting the results to Slack every week

The AI part is genuinely useful here. A permit tells you there's a $1.2M warehouse being built at 12500 Vulcan Way. It doesn't tell you whether that's a local job with a 5-person crew or a fly-in crew of 60. Claude can make a reasonable inference from the project type, value, and description — and flag the ones that are worth calling about first.

The next few posts are about how that actually gets built. Starting with the tech stack — which has some choices in it that I think are worth explaining.
