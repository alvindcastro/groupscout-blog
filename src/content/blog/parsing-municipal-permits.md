---
title: "Parsing municipal permits — the first signal"
description: "How I turned messy city PDF reports into structured leads and why a modular approach scales."
pubDate: "2026-04-11"
draft: false
---

To learn what a city builds, visit its website. Many cities publish weekly lists of issued building permits.

These are perfect data sources. A $20M commercial permit signals that a crew will soon mobilize.

The problem is that the data usually resides in a PDF, not an API.

---

## The Modular Strategy

Early on, I realized that a universal permit parser is a trap; every city uses different software, layouts, and descriptions.

I build modular collectors instead. Each city receives its own implementation to handle its quirks.

---

## The PDF Problem

PDFs are where data dies. They are for printing, not parsing.

A building permit report shows columns for folder numbers, addresses, work types, and construction values. A computer sees only characters at coordinates.

My first attempt to scrape the HTML preview failed because of nested tables and inconsistent formatting. The second attempt succeeded: I downloaded the PDF and used `pdftotext`.

---

## pdftotext to the rescue

Group Scout uses `pdftotext` from the Poppler suite. This tool converts a PDF to plain text while maintaining the layout.

In Go, a collector runs the command like this:

```go
cmd := exec.Command("pdftotext", "-layout", pdfPath, "-")
output, err := cmd.Output()
```

The `-layout` flag is essential; it preserves columns and makes it easy to write a parser that knows character positions.

---

## The "Fuzzy" Parser

Municipal data is messy: addresses span lines, names are truncated, and values might lack dollar signs.

The collector uses a line-by-line parser. It finds lines starting with a folder number and extracts fields.

It is not perfect, but it need not be. Claude reads the data next. Whether the parser extracts '1,200,000' or '$1.2M', Claude understands both. The parser merely places raw text into the right bucket.

---

## Why this is the "First Real Signal"

Generalizing permit parsing made Group Scout feel like a real product.

When the first Monday Slack message arrived with an automated lead, it validated the concept.

It proved:

1. The data exists.
2. The stack (Go, SQLite, and `pdftotext`) handles it.
3. The modular collector model scales across regions.

---

## What's next?

The permit collector pattern is stable. I am now adding sources and cities. Government contract awards are next; I must search thousands of tenders for those that matter.

I am pleased that messy municipal PDFs now work for me.
