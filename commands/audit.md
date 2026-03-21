---
description: Run a full storefront audit on a URL using acquire payload
allowed-tools: mcp__merch-connector__acquire, mcp__merch-connector__site_memory, mcp__merch-connector__ask_page, Write, Bash
argument-hint: <storefront-url>
---

Audit the storefront at $ARGUMENTS using merch-connector v2. The new `acquire` tool replaces scrape_page + audit_storefront, delivering a complete payload in one call. Follow these steps in order.

## Step 1 — Load prior context

Call `site_memory(action="read", url=$ARGUMENTS)`. Note:
- Prior dimension scores (if any) — you will highlight score changes in the report
- Any notes about rendering quirks, wait times, or known issues
- The `changes` object if present — flag price changes, new/removed products, facet changes

---

## Step 2 — Acquire complete page payload

Call `acquire(url=$ARGUMENTS, pdp_sample=2)`. This single call returns:
- Full page metadata (`title`, `metaDescription`, `pageType`, `h1`, `breadcrumb`)
- Commerce mode (`B2C` / `B2B` / `Hybrid`)
- All products with scores, prices, trust signals, B2B/B2C indicators
- Facets, sort options, and navigation signals
- Desktop + mobile screenshots (base64)
- Performance metrics (FCP, LCP, CLS, full load time)
- Trust signal aggregates across the page
- 2 pre-sampled PDP payloads (mid-range + premium)
- Warnings (NO_PRODUCTS_FOUND, MOBILE_RENDER_FAILED, etc.) with severity
- Site memory from prior audits

**Determine persona routing immediately from `payload.commerce.mode`:**
- `B2B` → use b2b_auditor lens (steps-to-PO, spec completeness, self-serve viability)
- `B2C` → use conversion_architect lens (CRO, social proof, funnel friction)
- `Hybrid` → use auditor lens, and flag mode conflict as a [STRATEGY] finding

**Check for critical warnings:**
- If `NO_PRODUCTS_FOUND` → STOP. Write report explaining bot block or wrong page type. Save memory note and exit.
- If `MOBILE_RENDER_FAILED` → add [ENGINEERING] finding; note mobile scoring unavailable.
- If `FIRECRAWL_FAILED` in warnings → add note "acquisition degraded, Firecrawl unavailable, scores may undercount description/rating quality."

---

## Step 3 — Optional: ask_page for payload gaps

If the payload is ambiguous on any specific UX or trust question, call `ask_page(url=$ARGUMENTS, question="<your question>")` once or twice. Good examples:
- "Does the filter panel collapse on mobile, and is it sticky?"
- "Are product ratings visible on the category card, or only on PDP?"

Most data is now in the payload; use this sparingly.

---

## Step 4 — Score using payload fields

Reference `payload.` fields explicitly when evaluating each dimension:

**UX & Design Quality (1–10):**
- `payload.navigation.hasFilterPanel`, `payload.navigation.filterPanelPosition`
- `payload.navigation.hasStickyNav`, `payload.navigation.hasSearchBar`
- `payload.navigation.breadcrumbPresent`
- `payload.performance.fcp` (>3000 ms = penalize), `payload.performance.loadComplete`
- `payload.facets.length` (0 = penalize heavily)
- Visual hierarchy from `payload.screenshots.desktop` and `.mobile`

**Conversion Optimization (1–10):**
- `payload.trustSignals.ratingsOnCards`, `payload.trustSignals.reviewsOnCards`
- `payload.trustSignals.freeShippingPromised`, `payload.trustSignals.returnPolicyVisible`
- `payload.trustSignals.urgencyMessaging` (present = good)
- `payload.analytics.productImpressionsFiring` (false = penalize high)
- `payload.dataQuality.ratingFillRate` (< 0.5 = issue)
- PDP data: `payload.pdpSamples[].reviewsVisible`, `payload.pdpSamples[].crossSellPresent`

**Copywriting & Messaging (1–10):**
- `payload.dataQuality.descriptionFillRate` (< 0.3 = penalize heavily)
- PDP data: `payload.pdpSamples[].descriptionWordCount` (< 50 = thin), `payload.pdpSamples[].descriptionIsTitle` = true → penalize
- `payload.page.h1` and `payload.page.breadcrumb` for messaging clarity
- Product `payload.products[].badges` quality and consistency

**Competitive Positioning (1–10):**
- `payload.commerce.platform` (detected e-commerce system)
- `payload.commerce.priceTransparency` (true = good)
- `payload.trustSignals.trustBadges` (count and relevance)
- `payload.page.metaDescription` quality and keyword alignment

---

## Step 5 — PDP drill-down (category pages only)

`payload.pdpSamples[]` contains 2 pre-scraped PDPs. Evaluate directly:
- `descriptionWordCount` < 50 → thin content [COPYWRITING]
- `specsPresent` = false → missing specs [DATA QUALITY]
- `crossSellPresent` = false → no cross-sell [STRATEGY]
- `reviewsVisible` = false → no social proof on PDP [CONVERSION]
- `imageCount` < 3 → insufficient imagery [UX DESIGN]

Summarize as a sub-section in the report. No additional scrape_pdp calls needed.

---

## Step 6 — Compile the audit report

Use the format below. Every observation must cite specific evidence — quote payload field names, reference actual values, or describe patterns from screenshots.

If prior scores exist in site_memory, add a delta next to each score: e.g. `7/10 (↑2 from last audit)`.

---

## Storefront Audit: $ARGUMENTS

**Audited:** [date] | **Commerce mode:** [payload.commerce.mode] | **Platform:** [payload.commerce.platform] | **Persona lens:** [b2b_auditor / conversion_architect / auditor]

### Summary
2–3 sentences. What does this storefront do well and where does it most need work? If this is a repeat audit, lead with what changed (cite `payload.changes` if present).

---

### 1. UX & Design Quality (score /10)
Evaluate: visual hierarchy, navigation clarity, mobile responsiveness, page load performance, facet usability, and sticky nav/search presence.

**Strengths:**
**Issues:** (tag each as `[ENGINEERING]`, `[UX DESIGN]`, `[FRONTEND INTEGRATION]`, or `[STRATEGY]`)
**Recommendations:**

---

### 2. Conversion Optimization (score /10)
Evaluate: trust signals on cards and PDPs, CTA clarity, pricing transparency, checkout indicators, urgency messaging, analytics tracking, and review/rating visibility.

**Strengths:**
**Issues:**
**Recommendations:**

---

### 3. Copywriting & Messaging (score /10)
Evaluate: headline clarity, product description quality and fill rate, value proposition, tone consistency, benefit vs. feature framing, and CTA copy clarity.

**Strengths:**
**Issues:**
**Recommendations:**

---

### 4. Competitive Positioning (score /10)
Evaluate: brand differentiation, pricing strategy signals, platform choice, unique selling points, and how clearly the brand communicates why to buy here.

**Strengths:**
**Issues:**
**Recommendations:**

---

### PDP Spot-Check (if applicable)
Summarize findings from `payload.pdpSamples[]`. Note description quality (word count, thin vs. substantive), image count, review presence, specs completeness, cross-sell presence, and gaps vs. listing card.

---

### Overall Score: /40

**Grade:** (A=35–40, B=28–34, C=20–27, D=<20)

### Top 5 Recommendations
Prioritized by estimated impact. Be specific — name the page element or copy pattern to change, and tag the owning team.

1.
2.
3.
4.
5.

---

## Step 7 — Save the report

Use Bash to resolve the workspace path and create the outputs directory:
```bash
WS=$(find /sessions/*/mnt -maxdepth 1 -type d -name "Merch-connector" ! -path "*local-plugins*" 2>/dev/null | head -1) && mkdir -p "$WS/outputs" && echo "$WS"
```

Get today's date: `date +%Y-%m-%d`

Extract the domain from the URL (e.g. `allbirds.com`, `insight.com`) and write the full audit report to `$WS/outputs/audit-[domain]-[YYYY-MM-DD].md`.

---

## Step 8 — Write findings back to site_memory

Call `site_memory(action="write", url=$ARGUMENTS, note="Audit [date]: UX=[score]/10, Conversion=[score]/10, Copy=[score]/10, Positioning=[score]/10, Total=[score]/40. Top issues: [1-sentence summary of top 3 findings].")`.

This ensures the next audit opens with prior context and can track score trends over time.
