---
title: "Claude scores hotel leads"
description: "A building permit identifies what was built, but Claude decides whether to call."
pubDate: "2026-04-05"
draft: false
---

A building permit lists a $1.2M warehouse build at 12500 Vulcan Way by Safara Cladding Inc for ABC Developments Ltd.

It omits:

- If "Safara Cladding" is the GC or a sub.
- The required crew size.
- If the crew is local or from out of province.
- If the lead warrants a call.

An industry expert infers these from project type, value, and location. Claude does the same. It adds context so a human can make a better call.

This isn’t about learning how to use an LLM — it’s about getting better at where it helps, where it doesn’t, and how much structure it actually needs to be useful.

---

## The enrichment call

Each permit passing the filter goes to the Claude Messages API with this system prompt:

```
You are a lead analyst for a hotel near a major airport.
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

Priority score, estimated crew, outreach timing, GC name, and a plain-English reason — everything a sales manager needs to decide on a call.

---

## Why Haiku and not Sonnet

Claude Haiku is faster and 5x cheaper than Sonnet. For structured extraction from short text, Haiku is the right choice. It reads a permit and fills the fields.

Permit enrichment costs roughly $0.001 per permit. The pipeline processes 5–10 permits weekly, totaling a few cents monthly.

---

## An important design choice: preserve the raw data

Claude _guesses_ the GC. I preserve the raw permit's `applicant` and `contractor` fields alongside Claude's inference.

Why does this matter? Take this example:

- **Raw applicant:** `ABC Developments Ltd`
- **Raw contractor:** `Safara Cladding Inc (416)875-1770`
- **Claude's GC inference:** `Safara Cladding Inc`

That phone number — `(416)875-1770` — is in the raw contractor field. If I only kept Claude's `general_contractor`, I'd lose the phone number. By preserving both, the sales manager has the raw contact info and Claude's interpretation of who's actually running the project.

The raw data never gets thrown away. Claude adds context on top of it — it doesn't replace it.

---

## Dedup before enrichment

Enrichment follows the dedup check. The pipeline checks if a permit's hash exists before calling Claude.

Claude calls cost money. Re-enriching the same permits would waste funds and generate duplicate leads and Slack notifications. Dedup prevents this.

The rule: if seen, skip. If new, enrich, store, and alert.

Next: the Slack digest.
