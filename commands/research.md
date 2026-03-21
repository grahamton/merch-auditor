---
description: Build a research dossier on any storefront or URL
allowed-tools: mcp__merch-connector__scrape_page, mcp__merch-connector__ask_page, mcp__merch-connector__site_memory, Write, Bash
argument-hint: <url> [optional: research question or focus area]
---

Build a research dossier on $ARGUMENTS using the merch connector.

This command is for open-ended site research — competitive analysis, client prep, category benchmarking, or any time you want to accumulate knowledge about a site without running a formal scored audit.

## Step 1 — Load existing memory

Call `site_memory(action="read", url=$ARGUMENTS)`. Summarize what's already known. If this is a first visit, note that.

## Step 2 — Scrape and orient

Call `scrape_page(url=$ARGUMENTS, include_screenshot=true, mobile_screenshot=true, max_products=20)`.

Produce a concise site profile:
- **What is this site?** (brand, category, commerce mode, estimated scale)
- **Platform signals** — detected commerce platform, GTM containers, analytics stack
- **Product catalog snapshot** — price range, brand mix, product count, SKU depth
- **Performance** — FCP, load time, mobile rendering status
- **Discovery quality** — facet count, sort options, search capability

## Step 3 — Answer the research question

If the user provided a specific focus area or question after the URL, use `ask_page` to address it directly. Otherwise, default to these high-value questions:
- "What is the primary value proposition communicated on this page? Quote the exact headline and any supporting copy."
- "Describe the checkout/conversion path visible from this page — what are the available CTAs and where do they lead?"

## Step 4 — Identify open questions

List 3–5 things you couldn't determine from the scrape that would be valuable to know — pages to visit next, questions to probe deeper, or data that requires login/interaction.

## Step 5 — Save the dossier

Use Bash to resolve the workspace path and create the outputs directory:
```bash
WS=$(find /sessions/*/mnt -maxdepth 1 -type d -name "Merch-connector" ! -path "*local-plugins*" 2>/dev/null | head -1) && mkdir -p "$WS/outputs" && echo "$WS"
```

Get today's date: `date +%Y-%m-%d`

Write findings to `$WS/outputs/research-[domain]-[YYYY-MM-DD].md`.

Format:

---
# Research: [Site Name]
**URL:** [url]
**Date:** [date]
**Focus:** [research question or "general"]

## Site Profile
[platform, commerce mode, catalog snapshot, performance]

## Key Findings
[bulleted findings from scrape + ask_page]

## Open Questions
[things to investigate next]

## Raw Notes
[anything else worth preserving]
---

## Step 6 — Write a note to site_memory

Call `site_memory(action="write", url=$ARGUMENTS, note="Research [date]: [2-sentence summary of what was learned].")`.

All future audits and research sessions on this domain will open with this context.
