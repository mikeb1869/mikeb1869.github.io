---
layout: post
title: "Math Workbook Generator: A Shared Engine Behind a 3-Book KDP Series"
image: "/posts/math-workbook-generator-banner.png"
date: 2026-07-17
tags: [python, pdf-generation, reportlab]
---

## What it is

A config-driven Python tool that generates print-ready book interiors for a series of elementary math workbooks I self-publish on Amazon KDP: Multiplication, Division, and Addition & Subtraction. Each book is around 100 pages, thousands of arithmetic problems, a guaranteed-correct answer key, and print specs that pass KDP's requirements: trim size, mirrored binding margins, embedded fonts.

Repo: [github.com/mikeb1869/math-workbook-generator](https://github.com/mikeb1869/math-workbook-generator)

## The problem I actually cared about

Generating a PDF from a Python script isn't the interesting part. The interesting part is that these three books are clearly the same product family: same layout language, same print specs, same series motifs (a stopwatch on the title page, an award seal on the certificate). But they also have real differences that can't be papered over: different operations, different audiences, different number ranges. A Grade 1-2 book needs bigger type and more writing room than a Grade 3-5 book. A book with 3-digit answers needs different column spacing than one with 2-digit answers.

I built the first book (Multiplication) as a single standalone script. When I built the second one (Division), I copy-pasted and adapted it, which is a completely reasonable way to move fast. By the third book, the three files had quietly drifted out of sync in ways that mattered, not just cosmetically.

## What drifting apart actually cost me

Pulling the shared logic into one `engine/` package and having all three books import it wasn't just a tidiness exercise. Doing it surfaced real bugs:

**A layout bug that only showed up on dense pages.** The function that draws each problem anchored the operator symbol to the left edge of a fixed-width column. That's fine when the column is narrow, but Division's dividends run to 3 digits, so a single-digit divisor like `7` would right-align in a column sized for `144`, leaving a big gap between the operator and its own number while the gap *between* adjacent problems stayed small. On a 100-problem page, the eye grouped the wrong digits together. I'd already fixed this once in Division's copy of the function. It never made it back into Multiplication's copy, because they were separate files.

**A missing guarantee.** Division and Addition & Subtraction both size their type so there's always a minimum amount of clear space under the line for a kid to actually write an answer. Multiplication's copy of that function predated the fix and never got it.

**A silent duplicate-problem bug.** The "Fact Families" pages (a differentiator I built into Division and Addition & Subtraction, teaching the inverse relationship between operations, e.g. 7 x 8 = 56 so 56 / 8 = 7) build four related facts from two numbers. If the two numbers happened to be equal, two of the four facts became identical, and the page would silently print a duplicate instead of four distinct problems.

None of these were things I'd have caught by reading the code casually. I caught them by treating "these three books should behave identically where they claim to" as something to actually check, then diffing the drift.

## What's shared, and what isn't

The full writeup is in the repo's README and CHANGELOG, but the short version: page layout, margins, the drawing primitives (the problem grid, the stopwatch motif, the certificate seal), and the answer-key renderer are shared, because they were either byte-identical or functionally identical across all three books before the refactor.

What stayed separate, on purpose: problem generation (each book's actual math and variety rules are genuinely different), front and back matter content (a multiplication table isn't a division facts table isn't an addition table), and the sizing constants passed into the shared page renderer. That last one matters. Grade 1-2 needing bigger type than Grade 3-5 is a real, audience-driven decision. Forcing it into one shared constant would have been the same mistake as the copy-paste drift, just in the other direction.

## Correctness by construction

Every problem is generated along with its answer. There's no separate "now go compute the key" step that could fall out of sync with the drill page. Division goes a step further: a problem is built *from* a clean quotient (`dividend = divisor * quotient`), so a remainder or a division by zero isn't just unlikely, it's structurally impossible.

```python
pages = multiplication.build_pages(CONFIG)
assert all(a * b == ans for p in pages for (a, b, sym, ans) in p["problems"])
```

All three books currently validate at zero answer errors across their full problem sets (Multiplication: 4,660 problems, Division: 4,130, Addition & Subtraction: 2,288).

## Stack

Python, ReportLab for PDF generation, no external dependencies beyond that. All three books run from a plain `python3 multiplication.py` with the config for each book living in a dict at the top of its file.
