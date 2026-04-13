---
title: 'Cracking the PDF — no API, no problem (mostly)'
description: 'Municipalities often publish building permits as weekly PDFs with no API. A pure Go PDF library returned zero rows. Then came the parser bug that looked like a config bug. Here is how all of it got fixed.'
pubDate: '2026-04-03'
draft: true
---

> This post focuses on one city’s permit PDFs, but these parsing problems are common to many “PDF‑as‑data” systems.

The target city for my first collector does not have a building permits API.

I knew this. I checked. There's a data portal and some datasets, but weekly building permit reports are PDFs at a known URL. To get the data, I scrape the page, download the PDF, and parse the text.

Fine. I've done worse.

---

## Step one: finding the PDF

The reports page has a consistent HTML structure. Each weekly report is a link to a PDF at a URL like:

```
/__shared/assets/buildingreport12345.pdf
```

A single regex finds it:

```go
var buildingReportRe = regexp.MustCompile(`/__shared/assets/buildingreport[^"]+\.pdf`)
```

That part worked. One HTTP GET and one regex gave me the latest report's URL. I download it to a temp file and parse.

---

## The PDF library that returned zero rows

I first tried `github.com/ledongthuc/pdf` — a pure Go PDF text extraction library. It's well-maintained and works on most PDFs.

The target city's PDFs are not most PDFs.

The library returned zero rows. No parsing error, no panic, no warning — just silence.

These PDFs use a non-standard font encoding that the library cannot decode. It cannot read the text. This common PDF problem has no clean pure-Go fix.

I had two options:
1. Find another pure Go library and hope it handles the encoding.
2. Use `pdftotext`.

`pdftotext`, part of the Poppler library, is battle-tested. It handles encoding edge cases and is bundled with Git for Windows.

```go
out, err := exec.Command(pdftotext, path, "-").Output()
```

It produces clean plain text. Every permit and every field is readable.

The lesson: don't fight encoding in a pure Go library. Use a tool that solved the problem twenty years ago.

---

## The parser bug that looked like a config bug

This build log entry matters because a bug can mislead you.

After switching to `pdftotext`, the collector parsed 20 permit records. None passed the $500,000 value filter. Every permit returned `ValueCAD = 1`.

I thought I misconfigured the threshold. I lowered it to $100,000, then $1. Still zero — but now I saw permits with values of $1.

Verbose logging showed every record had `ValueCAD = 1` or `ValueCAD = 0`. The parser was wrong.

I dumped the raw `pdftotext` output and found this:

```
25 036523 000 00 B7
Alteration
Issued
2026/03/16
1
CONSTR. VALUE
$300,000.00
FOLDER NAME 8640 Alexandra Road
Studio Senbel Architecture Inc
Safara Cladding Inc (416)875-1770
```

The bare `1` between the date and the dollar amount is a permit count row. The original parser assigned fields by position:

- Line 0: work proposed
- Line 1: status
- Line 2: issue date
- **Line 3: value** ← the `1` sits

The `$300,000.00` landed in the `Address` field. The contractor's phone number ended up in an unknown slot. Everything shifted, and `ValueCAD` became `1`.

---

## The fix: content-aware parsing

I stopped trusting line position and started trusting line content:

```go
// It's a date
if dateRe.MatchString(line) {
    record.IssueDate = line
    continue
}
// It's a dollar value
if dollarRe.MatchString(line) {
    if record.ValueCAD == 0 {
        record.ValueCAD = parseDollar(line) // first dollar = construction value
    }
    continue
}
// It's an address
if strings.HasPrefix(line, "FOLDER NAME ") {
    record.Address = strings.TrimPrefix(line, "FOLDER NAME ")
    continue
}
// Bare integer — permit count row, skip
if intRe.MatchString(line) {
    continue
}
// Everything else: sequential fill
```

This handles the clean test fixture and the `pdftotext` output. If a line looks like a dollar amount, it becomes `ValueCAD`. If it looks like a date, it goes to `IssueDate`. The bare `1` is skipped.

After the fix, 3 of 20 permits passed the filter with correct construction values.

---

## 32 unit tests

I wrote tests for every pure function. The parser is a pure function: give it lines, get a record. It should have test cases:

```go
{
    name: "real pdftotext output with count row",
    lines: []string{"Alteration", "Issued", "2026/03/16", "1", "CONSTR. VALUE", "$300,000.00", "FOLDER NAME 8640 Alexandra Road", ...},
    want: RawProject{ValueCAD: 300000, Address: "8640 Alexandra Road", ...},
},
```

I would have caught the bug immediately if I had used real `pdftotext` output in the test. I used clean data instead. Lesson noted.

Next: why I'm asking an AI to read a building permit.
