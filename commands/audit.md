---
description: Run a full storefront audit on a URL
allowed-tools: mcp__merch-connector__audit_storefront, mcp__merch-connector__scrape_page, mcp__merch-connector__ask_page, mcp__merch-connector__site_memory, Write, Bash
argument-hint: <storefront-url>
---

Audit the storefront at $ARGUMENTS using the merch connector. Follow these steps in order.

## Step 1 — Load prior context

Call `site_memory(action="read", url=$ARGUMENTS)`. Note:
- Prior dimension scores (if any) — you will highlight score changes in the report
- Any notes about rendering quirks, wait times, or known issues
- The `changes` object if present — flag price changes, new/removed products, facet changes

## Step 2 — Scrape the page

Call `scrape_page(url=$ARGUMENTS, include_screenshot=true, mobile_screenshot=true, max_products=20)`.

From the response, immediately determine **commerce mode**:
- If `fingerprint.commerceMode.mode` is `B2B` or `Hybrid` → set persona = `b2b_auditor`
- If `B2C` → set persona = `conversion_architect`
- If unclear → set persona = `auditor`

Note any rendering warnings (FCP=0, blank mobile screenshot, Unknown Facets) — these are pre-classified `[ENGINEERING]` findings.

## Step 3 — Probe with ask_page

Run 1–2 targeted `ask_page` questions based on what the scrape left ambiguous. Good defaults:
- Navigation structure, filter panel detail, and whether any content appears above the product grid
- Trust signals visible on product cards (ratings, badges, guarantees)

## Step 4 — PDP drill-down (category pages only)

If the audited page is a category/search results page (not a homepage or PDP), pick 2 products from the scrape results — ideally one mid-range and one premium — and call `scrape_page` on each PDP URL. Note:
- Description fill rate (is description = title, or is there real copy?)
- Image count
- Reviews/ratings present
- Cross-sell or upsell modules
- Spec completeness vs. the listing card

Summarize PDP findings as a sub-section in the report.

## Step 5 — Compile the audit report

Use the format below. Every observation must cite specific evidence — quote actual copy, reference field names from the scrape, or describe specific UI patterns from the screenshot.

If prior scores exist in site_memory, add a delta next to each score: e.g. `7/10 (↑2 from last audit)`.

---

## Storefront Audit: $ARGUMENTS

**Audited:** [date] | **Commerce mode:** [B2B / B2C / Hybrid] | **Persona lens:** [b2b_auditor / conversion_architect / auditor]

### Summary
2–3 sentences. What does this storefront do well and where does it most need work? If this is a repeat audit, lead with what changed.

---

### 1. UX & Design Quality (score /10)
Evaluate: visual hierarchy, navigation clarity, mobile responsiveness, page load performance (cite actual FCP/load numbers), whitespace, design system consistency, filter/facet usability.

**Strengths:**
**Issues:** (tag each as `[ENGINEERING]`, `[UX DESIGN]`, `[FRONTEND INTEGRATION]`, or `[STRATEGY]`)
**Recommendations:**

---

### 2. Conversion Optimization (score /10)
Evaluate: trust signals (reviews, guarantees, badges), CTA placement and clarity, pricing transparency, checkout friction indicators, urgency/scarcity use, social proof, analytics tracking gaps.

**Strengths:**
**Issues:**
**Recommendations:**

---

### 3. Copywriting & Messaging (score /10)
Evaluate: headline clarity, value proposition, product description quality (note fill rate), tone consistency, benefit vs. feature framing, CTA copy, page title/meta accuracy.

**Strengths:**
**Issues:**
**Recommendations:**

---

### 4. Competitive Positioning (score /10)
Evaluate: brand differentiation, pricing strategy signals, unique selling points, how clearly the brand communicates why to buy here vs. elsewhere.

**Strengths:**
**Issues:**
**Recommendations:**

---

### PDP Spot-Check (if applicable)
Summarize findings from the 2 PDP scrapes. Note description quality, image count, review presence, and any gaps vs. the listing card.

---

### Overall Score: /40

### Top 5 Recommendations
Prioritized by estimated impact. Be specific — name the page, element, or copy pattern to change, and tag the owning team.

1.
2.
3.
4.
5.

---

## Step 6 — Save the report

Use Bash to resolve the workspace path and create the outputs directory:
```bash
WS=$(find /sessions/*/mnt -maxdepth 1 -type d -name "Merch-connector" ! -path "*local-plugins*" 2>/dev/null | head -1) && mkdir -p "$WS/outputs" && echo "$WS"
```

Get today's date: `date +%Y-%m-%d`

Extract the domain from the URL (e.g. `allbirds.com`, `insight.com`) and write the full audit report to `$WS/outputs/audit-[domain]-[YYYY-MM-DD].md`.

## Step 7 — Write findings back to site_memory

Call `site_memory(action="write", url=$ARGUMENTS, note="Audit [date]: UX=[score]/10, Conversion=[score]/10, Copy=[score]/10, Positioning=[score]/10, Total=[score]/40. Top issues: [1-sentence summary of top 3 findings].")`.

This ensures the next audit opens with context and can track score trends over time.
