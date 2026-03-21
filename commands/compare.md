---
description: Compare two storefronts side by side (v2 workflow)
allowed-tools: mcp__merch-connector__site_memory, mcp__merch-connector__acquire, mcp__merch-connector__compare_storefronts, Write, Bash
argument-hint: <url-a> <url-b>
---

Compare two storefronts across structure, trust signals, commerce mode, and data quality. Parse the first URL as $1 (Site A) and the second as $2 (Site B).

Workflow:
1. `site_memory(read, domain=$1)` and `site_memory(read, domain=$2)` — load prior context for both
2. `acquire($1, pdp_sample=1)` → payload A
3. `acquire($2, pdp_sample=1)` → payload B
4. `compare_storefronts($1, $2)` → structural diff (product count delta, facet gap, trust signal coverage, performance delta)
5. Score both sites across 4 dimensions using the acquired payloads (see scoring signals below)
6. Save report to workspace outputs/ folder:
   ```bash
   WORKSPACE=$(cd "$(dirname "$0")/.." && pwd)
   DOMAIN_A=$(echo "$1" | sed 's|https\?://||;s|www\.||;s|/.*||')
   DOMAIN_B=$(echo "$2" | sed 's|https\?://||;s|www\.||;s|/.*||')
   DATE=$(date +%Y-%m-%d)
   mkdir -p "$WORKSPACE/outputs"
   # Write report to: $WORKSPACE/outputs/compare-$DOMAIN_A-vs-$DOMAIN_B-$DATE.md
   ```
7. `site_memory(write, domain=$1)` and `site_memory(write, domain=$2)` with findings

---

## Edge Case Handling

- If `payload.warnings[]` includes `NO_PRODUCTS_FOUND` for either site: note bot-blocking or wrong page type; skip scoring for that site and explain in report
- If `payload.screenshots.mobile === null` or `MOBILE_RENDER_FAILED`: note "[ENGINEERING] Mobile screenshot unavailable" in that site's UX section
- If `payload.scraper === 'puppeteer'` and `FIRECRAWL_FAILED`: note acquisition was degraded; data quality scores may undercount

---

## Scoring Signals by Dimension

**UX & Design** — `payload.navigation.hasFilterPanel`, `payload.navigation.filterPanelPosition`, `payload.navigation.hasStickyNav`, `payload.navigation.hasSearchBar`, `payload.navigation.breadcrumbPresent`, `payload.facets.length`, `payload.screenshots.mobile`

**Conversion Optimization** — `payload.trustSignals` (ratings, reviews, badges, stockWarning), `payload.dataQuality.ratingFillRate`, `payload.commerce.priceTransparency`, `payload.performance.fcp`, `payload.pdpSamples[].specsPresent`, `payload.pdpSamples[].crossSellPresent`

**Copywriting & Messaging** — `payload.dataQuality.descriptionFillRate`, `payload.page.metaDescription`, `payload.page.h1`, `payload.products[].description`, `payload.pdpSamples[].description`

**Competitive Positioning** — `payload.commerce.mode`, `payload.commerce.platform`, `payload.page.aboveTheFoldContent`, `payload.products[].badges`, `payload.products[].b2bIndicators`, `payload.products[].b2cIndicators`

---

## Storefront Comparison

**Site A:** $1
**Site B:** $2

---

### Dimension Scorecard

| Dimension | Site A | Site B | Winner |
|-----------|--------|--------|--------|
| UX & Design | /10 | /10 | |
| Conversion Optimization | /10 | /10 | |
| Copywriting & Messaging | /10 | /10 | |
| Competitive Positioning | /10 | /10 | |
| **Total** | **/40** | **/40** | |

---

### UX & Design
Layout, navigation, visual hierarchy, mobile responsiveness, facet panel, breadcrumb clarity. What does each do better?

**Site A:**
**Site B:**
**Winner:**

---

### Conversion Optimization
Trust signals (ratings, reviews, badges), CTA clarity, pricing transparency, stock visibility, shipping info, return policy prominence.

**Site A:**
**Site B:**
**Winner:**

---

### Copywriting & Messaging
Product copy fill rate, description quality, headline clarity, value proposition, tone consistency.

**Site A:**
**Site B:**
**Winner:**

---

### Competitive Positioning
Brand differentiation, unique selling points, commerce mode (B2C vs. B2B), target customer clarity, market positioning.

**Site A:**
**Site B:**
**Winner:**

---

### Overall Winner
Which storefront performs better overall and why? (2–3 sentences)

### What Each Should Steal From the Other

**Site A should adopt from Site B:**
-
-
-

**Site B should adopt from Site A:**
-
-
-
