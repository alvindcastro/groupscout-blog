---
title: "Parsing Municipal PDFs — Content-Aware Data Extraction in Go"
description: "Why pure Go PDF libraries fail on municipal encoding and how to build a robust, content-aware parser for inconsistent data."
pubDate: "2026-04-11"
tags: ["go", "pdf", "parsing", "automation"]
draft: false
---

Many municipalities lack a proper data API. They publish weekly building permits as PDFs. To automate data collection, the system must download the PDF, extract text, and parse it into structured records.

## The Encoding Problem

I first tried a pure Go PDF library. It works for standard PDFs but returns zero rows for certain municipal reports. These documents use non-standard font encoding that many lightweight libraries cannot decode.

Instead of fighting encoding edge cases in Go, I use `pdftotext` from the Poppler library. It handles these encodings reliably and is widely available.

```go
func extractText(path string) (string, error) {
    out, err := exec.Command("pdftotext", path, "-").Output()
    if err != nil {
        return "", err
    }
    return string(out), nil
}
```

This decision moved the complexity from my code to a specialized tool that has solved this problem for decades.

## From Position to Content

Initial parser versions assigned fields by line number. This approach is fragile. A single additional line—such as a permit count—shifts every subsequent field.

The parser now ignores position and evaluates content using regular expressions.

```go
func parseLines(lines []string) RawProject {
    var project RawProject
    for _, line := range lines {
        if dateRe.MatchString(line) {
            project.IssueDate = line
            continue
        }
        if dollarRe.MatchString(line) {
            if project.Value == 0 {
                project.Value = parseDollar(line)
            }
            continue
        }
        if strings.HasPrefix(line, "FOLDER NAME ") {
            project.Location = strings.TrimPrefix(line, "FOLDER NAME ")
            continue
        }
    }
    return project
}
```

This "content-aware" strategy handles variations in the PDF output. If a line looks like a date, it is a date. If it looks like a dollar value, it is a value. The system skips irrelevant rows—like permit counts—automatically.

## Testing with Real Data

I learned that testing with clean data is insufficient. I now use raw `pdftotext` output in test fixtures to catch the specific formatting quirks that occur in production.

A robust parser must expect noise. By focusing on content rather than position, the collector remains stable even when the PDF layout shifts.
