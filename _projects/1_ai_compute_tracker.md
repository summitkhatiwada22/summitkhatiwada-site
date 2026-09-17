---
layout: page
title: AI Compute Tracker
description: A free, fully-automated daily tracker of GPU pricing, hyperscaler capex, and AI funding — pulled straight from primary sources.
img: assets/img/project_images/ai-compute-tracker-cover.png
importance: 1
category: work
related_publications: false
---

[AI Compute Tracker](https://github.com/summitkhatiwada22/ai-compute-tracker) collects three data series about the AI infrastructure buildout, fully automated via GitHub Actions and updated daily:

- **GPU pricing** — marketplace (Vast.ai) and neocloud (Lambda Cloud) rental rates across all available GPU models
- **Hyperscaler capex** — capex, revenue, and operating cash flow for Microsoft, Amazon, Alphabet, Meta, Oracle, and CoreWeave, pulled directly from SEC EDGAR XBRL filings
- **AI sector funding** — Dealroom's public market-map data, tracked dynamically across every available tag

Everything is free to run (no paid APIs), and the [README](https://github.com/summitkhatiwada22/ai-compute-tracker) documents the real limitations of each data source rather than glossing over them.