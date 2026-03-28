---
title: 'Cracking the PDF — no API, no problem (mostly)'
description: 'Richmond publishes building permits as weekly PDFs with no API. A pure Go PDF library returned zero rows. Then came the parser bug that looked like a config bug. Here is how all of it got fixed.'
pubDate: '2026-03-31'
heroImage: '../../assets/blog-placeholder-5.jpg'
---

The City of Richmond does not have a building permits API.

I knew this going in. I checked. There's a data portal, there are some datasets, but weekly building permit reports are published as PDFs at a known URL. That's it. If you want the data, you scrape the page, download the PDF, and parse the text.

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

That part worked first try. One HTTP GET, one regex, and I have the URL of the latest report. Download it to a temp file. Now parse it.

---

## The PDF library that returned zero rows

My first attempt used `github.com/ledongthuc/pdf` — a pure Go PDF text extraction library. It's well-maintained, does what it says, works on most PDFs.

Richmond's PDFs are not most PDFs.

The library returned zero rows. Not a parsing error — just an empty result. No panic, no warning, just silence.

After some digging: Richmond's permit PDFs use a non-standard font encoding that the library can't decode. The text is there, but the library can't read it. This is a known class of PDF problem and there's no clean pure-Go fix for it.

At this point I had two options:
1. Find another pure Go PDF library and hope it handles the encoding
2. Shell out to `pdftotext`

`pdftotext` is part of the Poppler library — battle-tested, handles every encoding edge case, and it's bundled with Git for Windows (which I already have). One line:

```go
out, err := exec.Command(pdftotext, path, "-").Output()
```

That's it. Clean plain text output. Every permit, every field, readable.

The lesson: don't fight encoding issues in a pure Go library. Shell out to a tool that has solved this problem for twenty years.

---

## The parser bug that looked like a config bug

This is the part of the build log I most wanted to write about, because it's a good example of how a bug can send you in completely the wrong direction before you find it.

After switching to `pdftotext`, the collector ran successfully and parsed 20 permit records. Zero passed the value filter. I had the minimum threshold set at $500,000. Every permit was coming back with `ValueCAD = 1`.

My first thought: the threshold is wrong. Maybe I misconfigured it. I lowered it to $100,000. Still zero. I lowered it to $1. Still zero — but now at least I could see the permits passing the filter, all with a value of $1.

Added verbose logging to dump every parsed record. Every single one had `ValueCAD = 1` or `ValueCAD = 0`. This was not a threshold issue. Something was wrong with the parser.

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

See that bare `1` between the date and the dollar amount? That's a permit count row — the number of permits in that sub-type section. The original parser assigned fields by position:

- Line 0: work proposed
- Line 1: status
- Line 2: issue date
- **Line 3: value** ← this is where the `1` sits

The `$300,000.00` was landing in the `Address` field. The contractor's phone number was ending up in some unknown slot. Everything was off by one, and `ValueCAD` was `1` for every permit.

---

## The fix: content-aware parsing

The solution was to stop trusting line position and start trusting what the line looks like:

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

This handles both the clean test fixture format and the real `pdftotext` output. It doesn't matter where the dollar amount appears in the record — if it looks like a dollar amount, it gets assigned to `ValueCAD`. If it looks like a date, it goes to `IssueDate`. The bare `1` gets skipped entirely.

After that fix: 3 of 20 permits passing the value filter, all with correct construction values. The right permits. The right dollar amounts.

---

## 32 unit tests

One thing I did throughout all of this: write tests for every pure function as I went. The parser is a pure function — give it lines of text, get back a permit record. That's exactly the kind of thing that should have test cases:

```go
{
    name: "real pdftotext output with count row",
    lines: []string{"Alteration", "Issued", "2026/03/16", "1", "CONSTR. VALUE", "$300,000.00", "FOLDER NAME 8640 Alexandra Road", ...},
    want: RawProject{ValueCAD: 300000, Address: "8640 Alexandra Road", ...},
},
```

The parser bug would have been caught immediately if I'd written the test with real `pdftotext` output first. I didn't — I wrote it with clean fixture data. Lesson noted.

Next: what happens once the permits are parsed, and why I'm asking an AI to read a building permit.
