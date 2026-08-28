---
layout: post
title: "Lead Tracker: A B2B Lead Management App for Independent Used Car Dealers"
image: "/posts/lead-tracker-dms.png"
date: 2026-08-01
tags: [Flutter, Dart, Mobile Development, Firebase, MVVM]
categories: [Flutter, Mobile Development]
excerpt: "My first Flutter app — a lead-tracking tool for solo independent used car dealers, validated through forum research and built on 22 years of firsthand auto retail experience."
---

## The problem

Independent used car dealers running small operations — typically 10 to 25 vehicles on a lot — often manage sales leads entirely by memory: a mix of calls, texts, and emails with no central system and no reliable way to know who still needs a follow-up. Leads fall through the cracks not because dealers don't care, but because nothing is built for how small a "small business" actually is here.

I spent 18 years as a Buyer/Purchasing Manager in auto retail at CarMax before moving into product management, so I had a strong hypothesis about where the pain was. Rather than build on assumption alone, I validated it first.

**[Source on GitHub](https://github.com/mikeb1869/lead-tracker)** · **[Market research repo](https://github.com/mikeb1869/dealer-market-research)**

## Validating the problem first

Before writing any app code, I built a Python scraper targeting the NIADA Independent Dealer Forum and its CRM subforum on DealerRefresh, one of the industry's main community forums for independent dealers. I ran quantitative analysis on the scraped data with pandas, matplotlib, and wordcloud, then used the Anthropic API to do qualitative pain-point extraction across the corpus.

That research ruled out two ideas I'd initially considered — inventory sourcing (a structural supply problem, not a tooling one) and recall compliance (existing solutions already cover it, and there are liability concerns with building in that space as an outsider) — and pointed clearly at lead management for solo dealers as the underserved workflow worth building for.

## What it does

- **Lead capture** across multiple channels — phone, SMS, email, website, social media
- **One-tap contact** — call, text, or email a lead directly from its detail screen via `url_launcher`
- **Status tracking** — Fresh, Contacted, Closed, Lost, so a dealer can prioritize who to follow up with next
- **Follow-up log** — every contact attempt recorded with channel, response, and notes, stored as a Firestore subcollection under each lead
- **Real-time sync** — all data updates live across sessions via Firestore streams
- **Secure auth** — email/password sign-up and sign-in via Firebase Auth

## Technical overview

- **Framework:** Flutter / Dart
- **Architecture:** MVVM with a repository pattern — `UI (Screens) → ViewModels → Repositories → Firebase`, with repositories as the single source of truth for all Firestore reads and writes
- **State management:** `ValueNotifier`, with `StreamBuilder` handling the real-time follow-up subcollection data
- **Backend:** Firebase Authentication, Cloud Firestore
- **Dependency injection:** `get_it`
- **Screens:** Login, Signup, Home, Lead Detail, Follow-Up Log, Add Lead, Add Follow-Up, plus an `AuthGate` to route based on auth state

## A real mistake, and a real fix

Partway through development I accidentally committed `firebase_options.dart` — the file containing my Firebase project's API keys and config — directly into git history. Once I caught it, deleting the file in a new commit wasn't enough; the exposed keys were still sitting in every prior commit, retrievable by anyone who looked.

I fixed it properly: rewrote the repository's git history with `filter-branch` to strip the file from every commit, then rotated the exposed Firebase credentials. It's not a glamorous bug, but it's a real one — the kind of mistake that's easy to make once and that taught me to actually think about what belongs in `.gitignore` before the first commit, not after.

## Why this project mattered

Lead Tracker was where I learned the architectural patterns — MVVM, the repository pattern, dependency injection with `get_it` — that I carried into every Flutter project since, including [Helio](#), my B2C circadian rhythm app now live on the App Store and Google Play. It was also the first time I validated a product idea with real data before writing a line of app code, rather than assuming the problem based on instinct alone — a habit from my product management background that I plan to keep using.

## What's next

I'm continuing to build focused, single-purpose tools for small businesses — the same instinct behind Lead Tracker, applied to new niches where I have real domain knowledge to draw on.
