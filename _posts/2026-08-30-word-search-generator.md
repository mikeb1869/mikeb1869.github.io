---
layout: post
title: "Building a Word Search Generator That Refuses to Ship Broken Puzzles"
image: "/posts/word-search-generator-img.png"
date: 2026-07-20
categories: [projects, python]
tags: [python, generative, publishing, typography, testing]
excerpt: "A 1,000-line Python pipeline that turns a theme plan into a 172-page print-ready puzzle book — and the silent bugs that took months to surface."
---

I've spent the last stretch building a generator that produces a complete,
print-ready puzzle book: 133 puzzles, an answer key, front matter, contents,
172 pages, straight to PDF in about fifteen seconds.

The interesting part wasn't the grid algorithm. Placing words in a letter grid
is a solved problem you can write in an afternoon. The interesting part was
everything that let broken output *look* correct.

## What it does

The pipeline takes a plan of themes and a curated word corpus and produces a
finished book. Roughly a thousand lines of Python across eight modules:
selection, placement, filling, verification, pool validation, difficulty
profiles, text metrics, and the render pipeline that drives headless Chromium
over Jinja2 templates and print CSS.

Each puzzle has to satisfy a contract before it's accepted:

- exactly 20 words, 4 to 13 letters, no digits
- at most 5 long words, so the grid doesn't become a wall of overlaps
- no word contained inside another word in the same puzzle
- at most 3 words sharing a 3-letter prefix
- at most 3 words sharing a 5-character substring
- every word appears in the grid exactly once, proven by exhaustive scan
- no word repeats anywhere else in the same decade
- every word fits its column in the rendered word list

A puzzle that can't meet the contract is rejected outright rather than shipped
degraded. That turned out to matter, because most of those rules exist as
scar tissue.

## The verification bug that hid every other verification bug

The verifier scans every line of the grid in both directions and counts
matches. It compared against a baseline of 2, assuming a placed word is found
once forward and once in the reversed line.

That baseline was wrong. The scanner already enumerated each line in both
orientations, so a correctly placed word matched exactly **once**. The check
was comparing against a count no correct word could produce — which meant a
word that genuinely appeared **twice** in the grid sailed through silently.

Every puzzle generated before that fix had unverified duplicates.

Then, with duplicate detection finally working, `ABBA` started failing. And it
should: a reversed line containing a palindrome still contains it, so the
scanner legitimately sees it twice at the same position. The expected match
count depends on the word. Palindromes expect 2, everything else expects 1.

Two bugs, one nested inside the other, and the first was masking the second.

## Clustering: passing every check and still unsolvable

A puzzle from a roller-skating theme once drew thirteen words sharing the root
`SKATE`. No duplicates, no containment, correct word count — every constraint
green. The result was unsolvable in practice, because the grid was a wall of
near-identical letter runs and every candidate match looked like every other.

That produced the shared-root cap. A smaller version of the same failure, from
words beginning `THE`, produced the shared-prefix cap.

The root cap has a limitation I decided not to fix. It matches substrings of 5
or more characters, so a 4-letter root — `TAPE` across `TAPE HISS`, `BLANK
TAPE`, `MIX TAPE` — slips underneath it entirely. Lowering the threshold to 4
would bind far too aggressively across every pool, so it's handled editorially
and documented as a known trade-off rather than quietly left as a surprise.

## The bug class I didn't have a test for

Three separate defects shared one cause, and it's the thing I'd take to the
next project:

**Every validator I'd written asked whether a puzzle *generates*. Not one asked
whether it *fits on the page*.**

All three passed the full automated suite. All three were found by looking at
rendered pages.

### Measuring text is harder than it looks

Long entries in the word list were overflowing their column, wrapping to two
lines, and leaving a hole in the layout grid.

My first fix used character count as a width proxy. Wrong in both directions:
a 13-character entry can be wider than a 15-character one, because `M` and `W`
carry roughly twice the advance width of `I` and `L`. That approach would have
shrunk a hundred entries unnecessarily while missing the ones that actually
overflowed.

Second attempt: measure real advance widths from the font file with
`fontTools`. Better — except I measured the variable font at its default weight
of 400, when the stylesheet requests 500. That understates every string by
about 2%, which is enough to mark a wrapping word as safe.

Third attempt measured at the correct weight and matched every case I'd
observed. It still shipped one failure: an entry clearing the column by *nine
thousandths of an inch* wrapped anyway, because the browser hints glyphs and
rounds sub-pixels in ways the font tables don't capture.

The final fix wasn't more precision. It was a 2% safety margin on top of the
measurement. Some quantities you can compute exactly; some you can only bound.

That check now lives in the pool validator, so an overflowing word fails
validation instead of reaching a page.

### A number that was right and then quietly wasn't

Word count per puzzle was originally drawn from a 20–25 range. The word list
renders in four columns — and only 20 and 24 divide evenly, so 120 of 133
puzzles had a ragged final row. Including, ironically, every 25-word puzzle.

Fixing it at 20 also closed the last generation failure mode. 20 was already
the floor the selector would accept, so a pool that could reach 20 but not the
higher number it happened to draw would previously fail outright rather than
degrade.

### Four puzzles that were never there

For several builds the render summary reported four placeholder puzzles. All
133 pools existed and were valid.

The orchestrator locates each pool by slugging its theme title. Four files had
picked up `-70s` / `-80s` filename suffixes, added to avoid a collision that
didn't exist — the era directories already disambiguated them. The lookup
missed, and fell through to the placeholder path without complaint.

Found by reading a number in the build summary that had been sitting there,
wrong, for days.

## A tri-state flag that behaves like a delete

Corpus entries carry a confidence of `high`, `medium` or `flag`. Selection
admits the first two and silently excludes the third. `flag` was meant as "come
back to this."

An audit found twelve entries still sitting in it. Six had been reviewed and
approved much earlier — but the edits promoting them used exact-text
replacement that failed silently, so the approval never reached the file.

The trap is that a flagged word is *invisible to the engine*. An approval that
never landed is indistinguishable from a rejection, and every pool still worked
fine without those words, so nothing ever surfaced it. Every edit in the
pipeline now asserts on its own result rather than assuming the write landed.

## What I'd take away

**Validators encode the failures you've already imagined.** Mine were thorough
about generation and completely blind to layout, because generation was the
part I thought was hard.

**Silent exclusion is worse than loud failure.** The flag tier, the slug
mismatch, and the off-by-one all shared a shape: the system quietly produced
less than it should have, and produced something plausible anyway.

**Some things can only be checked by looking.** Every layout bug in this
project was found on a rendered page, not by a test. The fix isn't to stop
writing tests — it's to know which class of problem your tests structurally
cannot see, and to build the habit of actually looking at the output.

## Status and source

The interior is complete: 133 of 133 puzzles generating from the real corpus,
zero placeholders, 172 pages. The cover is in production with a designer, front
matter needs a final pass, and a physical proof is pending before it goes to
print.

**The source is in a private repository.** The engine is fairly generic, but
the corpus and the editorial framework that produced it represent the bulk of
the work and aren't something I'm publishing. Happy to walk through the
architecture or share code on request — [get in touch](/contact/).
