---
layout: post
title: "Dealer Market Research Pipeline: Scraping Forums + LLM Extraction to Decide What to Build"
image: "/posts/Dealer-Market-Research-Pipeline.png"
date: 2026-08-15
categories: [projects, python]
tags: [python, web-scraping, pandas, llm, market-research]
---

# Dealer Market Research Pipeline: Scraping Forums + LLM Extraction to Decide What to Build

Before writing any application code for what became Lead Tracker, I needed to answer a basic question: was there a real, solvable pain point for independent used-car dealers, or was I about to build something based only on my own assumptions?

I had spent 18 years in auto retail (including time as a Buyer/Purchasing Manager at CarMax), so I had hypotheses. I did not want to treat those hypotheses as facts. This project was the validation step.

## The goal

Identify recurring operational pain points in the independent used-car dealer space by analyzing real conversations, then decide which (if any) represented a viable point-solution opportunity worth building.

## What I built

A four-stage Python research pipeline:

1. **Data collection** — Scraped publicly accessible threads from two DealerRefresh subforums (NIADA Independent Dealer Forum and the CRM / ILM / DMS subforum).
2. **Quantitative analysis** — Keyword frequency, category coverage, and post-volume trends across the collected corpus.
3. **Qualitative extraction** — Batched post content and sent it through the Anthropic API (Claude) with a structured schema to pull out individual pain points.
4. **Synthesis** — Aggregated the extracted pain points, ranked them by severity and workaround density, and produced a clear product recommendation.

### Corpus size
- 835 posts
- 131 threads
- 233 unique authors
- 336 structured pain points extracted

## Technical overview

**Stack**
- Python
- `requests` + BeautifulSoup (scraping)
- pandas, matplotlib, wordcloud (quantitative analysis and visualization)
- Anthropic SDK / Claude (structured qualitative extraction)

**Pipeline scripts**
- `dealerrefresh_scraper.py` — Collects threads and posts
- `analyze_posts.py` — Quantitative analysis and chart generation
- `pain_point_analyzer.py` — Sends batches to Claude and extracts structured pain points
- `summarize_pain_points.py` — Aggregates, ranks, and surfaces the highest-signal opportunities

Each extracted pain point carried:
- Description
- Workflow affected
- Severity (high / medium / low)
- Verbatim signal phrase
- Any workaround the dealer mentioned

## Key findings

### Pain points by workflow category

| Category              | Pain Points | High Severity |
|-----------------------|-------------|---------------|
| Software & Tools      | 96 (28.6%)  | 56            |
| Lead Follow-up        | 52 (15.5%)  | 32            |
| Operations            | 48 (14.3%)  | 21            |
| Reporting             | 36 (10.7%)  | 14            |
| Financing             | 30 (8.9%)   | 22            |
| Communication         | 30 (8.9%)   | 15            |
| Compliance            | 24 (7.1%)   | 18            |
| Inventory Sourcing    | 20 (6.0%)   | 8             |

### Opportunities that emerged

Three stood out after ranking by severity and density of workarounds:

- **Lead management for solo operators** — One person handling all leads manually with no affordable CRM that matched the scale of a 10–25 car lot.
- **Recall compliance tracking** — Dealers checking VINs one by one; existing automated solutions were either too expensive or not aimed at independents.
- **Inventory sourcing** — Frequently mentioned, but the research showed it was primarily a structural supply problem rather than a tooling problem. I eliminated it as a product opportunity.

## The decision

Lead management for solo independent dealers was the clearest fit: high severity, frequent mentions, existing workarounds that were clearly inadequate, and a scope small enough to be tackled as a focused mobile tool.

That decision became [Lead Tracker](https://github.com/mikeb1869/lead-tracker).

## Why this project matters to me

This was the first time I treated market research as an engineering problem instead of a slide deck. The pipeline forced me to be specific about what “pain” actually looked like in the data, and it prevented me from building the wrong thing.

It also reinforced a habit I want to keep: validate with real evidence before writing product code.

## Source

The full pipeline, analysis scripts, and generated charts are public:

**[github.com/mikeb1869/dealer-market-research](https://github.com/mikeb1869/dealer-market-research)**

The Flutter app that resulted from this research is here:  
[github.com/mikeb1869/lead-tracker](https://github.com/mikeb1869/lead-tracker)
