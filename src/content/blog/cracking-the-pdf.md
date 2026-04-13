---
title: "Cracking the PDF — no API, no problem (mostly)"
description: "Municipalities often publish building permits as PDFs without an API. A pure Go PDF library failed, and a parser bug masqueraded as a configuration error. Here is how I fixed them."
pubDate: "2026-04-05"
draft: false
---

> This post focuses on one city’s permit PDFs, but these parsing problems are common to many “PDF‑as‑data” systems.

The target city for my first collector lacks a building permits API.

I checked the data portal and found datasets, but weekly reports remain PDFs at a known URL. To get the data, I scrape the page, download the PDF, and parse the text.

Fine. I've done worse.

---

## Step one: finding the PDF

The reports page has a consistent HTML structure; each weekly report links to a PDF at a URL like:

```
/__shared/assets/buildingreport12345.pdf
```

A single regex finds it:

```go
var buildingReportRe = regexp.MustCompile(`/__shared/assets/buildingreport[^"]+\.pdf`)
```

One HTTP GET and one regex provided the URL. I download the report to a temporary file and parse it.

---

## The PDF library that returned zero rows

I first tried `pdf`, a pure Go text extraction library. It works on most PDFs.

The target city's PDFs are not most PDFs.

The library returned zero rows. No parsing error, no panic, no warning — just silence.

The library cannot decode the non-standard font encoding used in these PDFs. This common problem lacks a pure Go solution.

I had two options:

1. Find another pure Go library and hope it handles the encoding.
2. Use `pdftotext`.

`pdftotext`, part of the Poppler library, handles encoding edge cases and is bundled with Git for Windows.

```go
out, err := exec.Command(pdftotext, path, "-").Output()
```

It produces clean text; every permit and field is readable.

Lesson: do not fight encoding in a pure Go library. Use a tool that solved the problem twenty years ago.

---

## The parser bug that looked like a config bug

A bug can mislead you.

After I switched to `pdftotext`, the collector parsed twenty records. None passed the $500,000 filter; every permit returned `ValueCAD = 1`.

I suspected a misconfigured threshold. I lowered it to $100,000, then to $1. The results remained zero, though I saw permits with values of $1.

Verbose logging showed that every record had `ValueCAD = 1` or `ValueCAD = 0`. The parser was failing.

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

The `1` between the date and the dollar amount is a permit count row. The original parser assigned fields by position:

- Line 0: work proposed
- Line 1: status
- Line 2: issue date
- **Line 3: value** ← the `1` sits

The `$300,000.00` landed in the `Address` field, and the phone number moved to an unknown slot. Everything shifted; `ValueCAD` became `1`.

---

## The fix: content-aware parsing

I stopped trusting position and started trusting content:

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

This handles both the test fixture and the `pdftotext` output. If a line resembles a dollar amount, it becomes `ValueCAD`. If it resembles a date, it becomes `IssueDate`. The `1` is skipped.

After the fix, three of twenty permits passed the filter.

---

## 32 unit tests

I wrote tests for every pure function. The parser is such a function: it receives lines and returns a record.

```go
{
    name: "real pdftotext output with count row",
    lines: []string{"Alteration", "Issued", "2026/03/16", "1", "CONSTR. VALUE", "$300,000.00", "FOLDER NAME 8640 Alexandra Road", ...},
    want: RawProject{ValueCAD: 300000, Address: "8640 Alexandra Road", ...},
},
```

I would have caught the bug immediately with real `pdftotext` output. I used clean data instead. Lesson noted.

Next: why I ask an AI to read a building permit.
