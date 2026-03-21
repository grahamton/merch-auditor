---
description: Build a research dossier on any storefront or URL
allowed-tools: mcp__merch-connector__acquire, mcp__merch-connector__site_memory, Write, Bash
argument-hint: <url> [optional: research question or focus area]
---

Build a research dossier on $ARGUMENTS using the merch connector v2.

This command is for open-ended site research — competitive analysis, client prep, category benchmarking, or any time you want to accumulate knowledge about a site without running a formal scored audit.

## Step 1 — Load existing memory

Call `site_memory(action="read", url=$ARGUMENTS)`. Summarize what's already known. If this is a first visit, note that.

## Step 2 — Acquire page data

Call `acquire(url=$ARGUMENTS, pdp_sample=1)`.

This returns a rich payload containing:
- `page` — title, metaDescription, breadcrumb, h1
- `commerce` — mode (B2C/B2B/Hybrid), platform, priceTransparency
- `products` — title, price, rating, reviewCount, badges, trustSignals
- `facets`, `sort`, `performance`, `trustSignals`, `analytics`, `navigation`
- `dataQuality` — productCount, descriptionFillRate, ratingFillRate
- `pdpSamples` — light PDP inspection (1 sample requested)
- `changes` — if prior baseline exists, delta vs. last scrape
- `siteMemory` — prior notes and scrape count

## Step 3 — Build site profile

From the payload, assemble a concise profile:
- **Brand & commerce mode** — from `payload.commerce.mode`, `payload.commerce.platform`
- **Catalog snapshot** — `payload.dataQuality.productCount`, top 3 products from `payload.products[]`
- **Performance** — `payload.performance.fcp`, `payload.performance.loadComplete`
- **Analytics stack** — `payload.analytics.platforms`, GTM containers
- **Trust signals** — summary from `payload.trustSignals`
- **Navigation quality** — facets (`payload.facets.length`), `payload.navigation.hasFilterPanel`, `payload.navigation.hasSearchBar`
- **Data quality flags** — surface any `payload.warnings[]` entries

## Step 4 — Answer the research question

If the user provided a specific focus area, extract relevant fields from the payload to address it. Otherwise, default to:
- **Value proposition** — from `payload.page.h1`, `payload.page.metaDescription`, top product titles and badges
- **Conversion path** — CTAs in product cards, `payload.navigation.hasSearchBar`, facets available for narrowing

## Step 5 — Identify open questions

List 3–5 things the payload doesn't cover or worth investigating next:
- PDP depth (reviews, specs, cross-sell — accessible via pdpSamples)
- Checkout flow (requires interaction, not in acquire)
- Login/account features (if `payload.commerce.loginRequired` is true)
- Historical changes (site_memory.scrapeCount — how often does this site update?)

## Step 6 — Save the dossier

Use Bash to resolve the workspace path:
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
[platform, commerce mode, catalog snapshot, performance, analytics]

## Key Findings
[profile + research question answer, bulleted]

## Data Quality
[fill rates, warnings, confidence flags]

## Open Questions
[things to investigate next]

---

## Step 7 — Write a note to site_memory

Call `site_memory(action="write", url=$ARGUMENTS, note="Research [date]: [2-sentence summary of what was learned].")`.

All future audits and research sessions on this domain will open with this context.
