Building a universal parser for municipal data is a trap.

Every city uses different software and layouts for their building permits. When I started Group Scout, I realized I could not build one parser for every city.

Instead, I built a modular collector system:
1. One implementation per city. I handle specific quirks without breaking the rest of the pipeline.
2. `pdftotext -layout`. This tool from the Poppler suite reliably extracts text while preserving columns.
3. The 'Fuzzy' Parser. My Go code puts the raw text into the right bucket. I let Claude handle the rest.

The moment the first automated lead reached my Slack channel, I knew the concept worked. Messy municipal PDFs are now a structured goldmine.

Full breakdown: [Link]
