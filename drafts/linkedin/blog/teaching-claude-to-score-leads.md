A building permit for a $1.2M warehouse is just raw data. It does not tell a hotel sales manager if the crew warrants a call.

Group Scout uses Claude to provide context.

We pass raw permit data to Claude Haiku. The LLM then acts as a lead analyst, returning a structured JSON object with:
1. Estimated crew size and project duration.
2. Likelihood of an out-of-town crew.
3. A priority score and rationale.
4. Suggested outreach timing.

I use Claude Haiku because it is fast, cheap, and accurate for structured extraction. At $0.001 per permit, it adds massive value.

One rule: Never discard raw data. Claude might infer the General Contractor, but I keep the original fields. If the raw data includes a phone number, it remains available.

AI provides the context to act on a lead.

Full story: [Link]
