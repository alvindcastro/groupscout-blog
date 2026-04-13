---
title: "Claude scores hotel leads"
description: "A building permit identifies what was built, but Claude decides whether to call."
pubDate: "2026-04-05"
draft: false
---

A building permit lists a $1.2M warehouse build at 12500 Vulcan Way by Safara Cladding Inc.

It omits:

- Whether Safara Cladding is the contractor or a subcontractor.
- The crew size.
- Whether the crew is local or from out of province.
- Whether the lead warrants a call.

An industry expert infers these details from the project type, value, and location; Claude does the same. It adds context to help a human decide.

This is about determining where an LLM helps, where it does not, and how much structure it requires.

---

## The enrichment call

Each permit passing the filter goes to the Claude Messages API with this system prompt:

```
You are a lead analyst for a hotel near a major airport.
Evaluate building permit records to identify projects that will generate
demand for construction crew lodging.
```

The permit data—folder number, address, type, value, applicant, contractor, and date—goes into the message. Claude returns a JSON object:

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

Priority, crew size, timing, contractor, and reason—these details help a sales manager decide whether to call.

---

## Why Haiku and not Sonnet

Claude Haiku is faster and cheaper than Sonnet. For structured extraction, Haiku is the right choice; it reads a permit and fills the fields.

Enrichment costs $0.001 per permit. The pipeline processes five to ten permits weekly, totaling a few cents monthly.

---

## Preserve raw data

Claude guesses the contractor. I preserve the raw `applicant` and `contractor` fields alongside Claude's inference.

Why does this matter? Take this example:

- **Raw applicant:** `ABC Developments Ltd`
- **Raw contractor:** `Safara Cladding Inc (416)875-1770`
- **Claude's GC inference:** `Safara Cladding Inc`

The phone number is in the raw contractor field. If I kept only the inference, I would lose the number. By preserving both, the sales manager has the raw contact information and the AI interpretation.

I never discard raw data. Claude adds context but does not replace the original information.

---

## Dedup before enrichment

Enrichment follows the deduplication check. The pipeline checks for a permit's hash before calling Claude.

Claude calls cost money. Re-enriching permits would waste funds and generate duplicates. Deduplication prevents this.

The rule: if seen, skip. If new, enrich, store, and alert.

Next: the Slack digest.
