# Connectors

## Required: merch-connector >=2.0.0

This plugin requires the **merch-connector** MCP server (v2.0.0+) to be active in your Cowork session. It powers all storefront acquisition, comparison, and memory capabilities.

| Capability | Tool | Notes |
|------------|------|-------|
| Acquire complete storefront payload | `acquire` | Primary tool — replaces `scrape_page` + `audit_storefront` in one call |
| Compare two storefronts | `compare_storefronts` | Structural diff: product count, facets, trust signals, performance |
| Recall and write site analysis | `site_memory` | Historical context, scores, notes |
| Ask questions about a page | `ask_page` | Available for supplemental queries if needed |
| Browse a product detail page | `scrape_pdp` | Direct PDP access (acquire handles this automatically via `pdp_sample`) |

> **v1 tools retired in v2:** `audit_storefront` has been removed from merch-connector. `scrape_page` is deprecated — use `acquire` instead.

## Setup

Ensure merch-connector v2.0.0 or higher is installed and connected before using any commands. If you see "tool not found" errors for `acquire`, your connector version is too old — upgrade to v2.

## acquire payload reference

The `acquire` tool returns a complete payload in a single call:

| Field | Description |
|-------|-------------|
| `payload.products[]` | Product cards with title, price, trustSignals, badges, b2bIndicators |
| `payload.facets[]` | Filter panel: name, type, options, selectedCount |
| `payload.commerce` | mode (B2C/B2B/Hybrid), platform, priceTransparency |
| `payload.dataQuality` | descriptionFillRate, ratingFillRate, imageFillRate |
| `payload.trustSignals` | sitesWideRating, reviewCount, badges, returnPolicy |
| `payload.analytics` | GTM containers, dataLayer events |
| `payload.performance` | FCP, totalLoad |
| `payload.screenshots` | desktop (base64), mobile (base64 or null) |
| `payload.pdpSamples[]` | PDP details: specsPresent, crossSellPresent, hasVideo, rating |
| `payload.warnings[]` | Structured quality flags (see below) |
| `payload.navigation` | hasFilterPanel, hasStickyNav, hasSearchBar, breadcrumbPresent |
| `payload.page` | title, h1, metaDescription, breadcrumb, aboveTheFoldContent |

### Warning codes → finding tags

| Warning code | Finding tag | Owner |
|---|---|---|
| `NO_PRODUCTS_FOUND` | `[ENGINEERING]` | Bot blocking or wrong page type |
| `MOBILE_RENDER_FAILED` | `[ENGINEERING]` | Mobile screenshot unavailable |
| `ECOMMERCE_TRACKING_GAP` | `[FRONTEND INTEGRATION]` | GTM/dataLayer not firing |
| `LOW_DESCRIPTION_FILL` | `[DATA QUALITY]` | Product copy missing from feed |
| `LOW_CARD_CONFIDENCE` | `[DATA QUALITY]` | Product card extraction uncertain |
| `FCP_ZERO` | `[ENGINEERING]` | Performance data unavailable |
| `FIRECRAWL_FAILED` | note in report | Acquisition degraded to Puppeteer |
