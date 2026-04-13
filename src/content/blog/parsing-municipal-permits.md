---
title: 'Parsing municipal permits — the first signal'
description: "How I turned messy city PDF reports into structured leads and why a modular approach scales."
pubDate: '2026-04-07'
draft: true
---

To know what a city builds, visit the municipal website. Many cities publish weekly lists of recently issued building permits.

They are perfect data sources for Group Scout. A $20M commercial permit signals that a crew will soon mobilize.

The problem? The data usually resides in a PDF, not a clean API.

---

## The Modular Strategy

Early on, I realized a "universal" permit parser is a trap. Every city uses different software, column layouts, and descriptions.

I build modular collectors instead. Each city gets its own `Collector` implementation to handle its quirks.

---

## The PDF Problem

PDFs are where data dies. They are for printing, not parsing.

A building permit report shows columns: Folder Number, Address, Work Type, and Construction Value. A computer sees only characters at X/Y coordinates.

My first attempt tried to scrape the HTML preview. It was a mess of nested tables and inconsistent formatting. The second attempt succeeded: I downloaded the PDF and used `pdftotext`.

---

## pdftotext to the rescue

Group Scout uses `pdftotext` from the Xpdf/Poppler suite. This battle-tested CLI tool converts a PDF to plain text while attempting to maintain the physical layout.

In Go, a collector runs the command like this:

```go
cmd := exec.Command("pdftotext", "-layout", pdfPath, "-")
output, err := cmd.Output()
```

The `-layout` flag is essential. It preserves columns, making it easy to write a parser that knows the Construction Value's character positions.

---

## The "Fuzzy" Parser

Municipal data is messy. Addresses span lines, contractor names are truncated, and construction values might lack dollar signs.

The collector uses a line-by-line parser. It finds lines starting with a folder number (e.g., `OF 24-123456`) and extracts relative fields.

It isn't perfect, but it needn't be. Claude reads the data next. If the parser extracts "1,200,000" or "$1.2M", Claude understands both. The parser just puts raw text into the right bucket.

---

## Why this is the "First Real Signal"

Generalizing permit parsing made Group Scout feel like a real product.

When the first Monday Slack message arrived with an automated lead — a permit for a warehouse renovation — it validated the concept.

It proved:
1. The data exists.
2. The stack (Go, SQLite, and `pdftotext`) handles it.
3. The modular collector model scales across regions.

---

## What's next?

The permit collector pattern is stable. Now I am adding more sources and cities. Government contract awards are next. I must search thousands of tenders for those that matter for hotel rooms.

I am happy I made messy municipal PDFs work for me.
