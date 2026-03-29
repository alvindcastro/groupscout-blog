---
title: 'Teaching Claude to score hotel leads'
description: "A building permit tells you what got built. It does not tell you whether to call someone about it. That is Claude's job."
pubDate: '2026-04-01'
draft: true
heroImage: '../../assets/blog-placeholder-1.jpg'
---

A building permit tells you: a $1.2M warehouse is being built at 12500 Vulcan Way, issued to Safara Cladding Inc, filed by ABC Developments Ltd.

What it doesn't tell you:
- Whether "Safara Cladding" is the GC or a sub
- How many workers the project will require
- Whether those workers are local or flying in from out of province
- Whether it's worth calling about at all

A human who knows the construction industry can make reasonable inferences from project type, value, and location. That's exactly what Claude is for.

---

## The enrichment call

Each permit that passes the initial filter gets sent to the Claude Messages API with a system prompt that establishes the context:

```
You are a lead analyst for a hotel near Vancouver International Airport.
Evaluate building permit records to identify projects that will generate
demand for construction crew lodging.
```

The permit data — folder number, address, work type, construction value, applicant name, contractor name, issue date — goes into the user message. Claude returns a JSON object:

```json
{
  "general_contractor": "BuildRight Contracting",
  "project_type": "industrial",
  "estimated_crew_size": 80,
  "estimated_duration_months": 6,
  "out_of_town_crew_likely": true,
  "priority_score": 9,
  "priority_reason": "Large new industrial build near YVR — likely out-of-province steel crew",
  "suggested_outreach_timing": "Reach out now — crews mobilizing in 4–6 weeks",
  "notes": "GC is BuildRight Contracting. Check LinkedIn for travel coordinator."
}
```

That's the lead. Priority score, estimated crew, outreach timing, GC name, and a plain-English reason. Everything a sales manager needs to decide whether to pick up the phone.

---

## Why Haiku and not Sonnet

Claude has a family of models. Sonnet is more capable. Haiku is faster and about 5x cheaper.

For this use case, Haiku is the right choice. The task is structured extraction from short text — not nuanced writing, not complex reasoning, just: read a permit, fill in these fields. Haiku handles it well.

The cost per permit is roughly $0.001 — one-tenth of a cent. The pipeline processes maybe 5–10 filtered permits per week. Total enrichment cost: a few cents per month. You could run this for years before the API cost becomes a rounding error worth caring about.

---

## An important design choice: preserve the raw data

One thing I was careful about: Claude's `general_contractor` field is its *best guess* at who the GC is. The raw permit has separate `applicant` and `contractor` fields — and those are preserved in the lead record alongside Claude's inference.

Why does this matter? Take this example:

- **Raw applicant:** `ABC Developments Ltd`
- **Raw contractor:** `Safara Cladding Inc (416)875-1770`
- **Claude's GC inference:** `Safara Cladding Inc`

That phone number — `(416)875-1770` — is in the raw contractor field. If I only kept Claude's `general_contractor`, I'd lose the phone number. By preserving both, the sales manager has the raw contact info and Claude's interpretation of who's actually running the project.

The raw data never gets thrown away. Claude adds context on top of it — it doesn't replace it.

---

## Dedup before enrichment

One more thing about where the enrichment fits in the pipeline: it happens *after* the dedup check, not before.

Before calling Claude, the pipeline checks whether this permit's hash already exists in the database. If it does, skip — regardless of whether enrichment previously succeeded or failed. (Failed enrichments get a retry path via the `enriched_at` flag, as described in the previous post.)

This matters because Claude API calls have a cost. Even at $0.001 per call, re-enriching the same 10 permits every time the pipeline runs would add up — and more importantly, would generate duplicate leads and duplicate Slack notifications. The dedup check keeps all of that clean.

The rule is simple: if you've seen it, skip it. If you haven't seen it, enrich it, store it, and tell someone about it.

Next post: the telling-someone-about-it part — the Slack digest.
