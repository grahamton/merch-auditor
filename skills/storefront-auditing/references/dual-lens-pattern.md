# Dual-Lens Pattern: API vs. UI Cross-Reference

The most actionable audit findings come from the gap between what a site's backend *has* and what its frontend *shows*. This reference documents the pattern and how to execute it.

## Why This Matters

An engineering team can fix a frontend integration gap in a sprint. A UX design problem might require a roadmap item. A data quality problem requires a catalog/merchandising fix. Conflating these into a single "filter labels are broken" finding wastes everyone's time. The dual-lens pattern separates them.

## Step-by-Step Execution

### Step 1: Scrape with full network capture
```
scrape_page(url, include_screenshot=true, mobile_screenshot=true)
```
Examine the response for:
- `networkIntel.platforms` — detected commerce search platform (Algolia, Coveo, SFCC, Elasticsearch, etc.)
- `networkIntel.apiCount` — how many API calls were intercepted
- `facets` array — raw facet names and option counts as extracted from the DOM
- `fingerprint.discoveryQuality` — facetCount, hasSearch, facetNames

### Step 2: Check the data layer
In `networkIntel.dataLayer`:
- Are ecommerce events firing? (`ecommerceEvents`, `totalProductImpressions`)
- Is user segment data present? (`userSegment`, authentication state)
- What GTM containers and GA properties are active?

Missing ecommerce events = conversion tracking is broken. This is a `[FRONTEND INTEGRATION]` finding with direct analytics impact.

### Step 2.5: Intercept the search API via browser (when scraper facets are sparse)

If the scrape returns few or unresolved facets, use Chrome MCP to intercept the actual search API the site is calling. This reveals the *full* facet schema the backend has — and is the ground truth for how many filters should be rendering.

**Why this matters:** Some sites (e.g. insight.com) use SPA architectures where browse pages fire no XHR on initial load — products are SSR'd, so the scraper sees no product search API calls. The real API only fires when a user interacts with filters. The scraper result is incomplete, not authoritative.

**Procedure:**

1. Navigate to the category page in Chrome and clear network request tracking:
```
navigate(url)
read_network_requests(clear=true)
```

2. Click a filter option to trigger the search API:
```
interact_with_page("Click the first available filter option in the left panel")
read_network_requests()
```
Look for XHR requests to a product search endpoint (e.g. `/gapi/product-search/search`, `/api/search`, `*.algolia.net/indexes/*/query`).

3. Once the API endpoint is identified, call it directly from the browser using `javascript_tool` to get the full facet payload:
```javascript
// Example: insight.com browse page API call
fetch('/gapi/product-search/search?' + new URLSearchParams({
  searchType: 'category',
  category: 'hardware/monitors',  // use the category code from the URL
  salesOrg: '2400',
  userSegment: 'CES',
  rows: '1',
  start: '0',
  lang: 'en_US',
  locale: 'en_US'
}))
.then(r => r.json())
.then(data => {
  window.__facetAudit = {
    totalFacets: data.facets ? data.facets.length : 0,
    facetNames: data.facets ? data.facets.map(f => f.name) : [],
    totalProducts: data.numFound
  };
});
'fired'
```
Then read the result: `javascript_tool("return JSON.stringify(window.__facetAudit)")`

> **Important note on async JS:** The `javascript_tool` does not support top-level `await`. Use `.then()` promise chains with a sentinel return value (`'fired'`) and read `window.__result` in a subsequent call.

> **Important note on query construction for browse pages:** On catalog/SPA sites, the search API often has different behavior depending on how the query is built. For insight.com browse pages, the facet/filter pipeline only fires correctly when `searchType=category` and the correct `category` code are included. Without these, the API may return results but with degraded or empty facets — this is correct behavior from the search engine's perspective (the merchandising rules require browse-mode query construction). **Do not classify this as a frontend integration bug.** It is a query construction issue: the UI must build browse queries correctly to fire the proper search pipeline and surface all available facets.

### Step 3: Validate rendered UI against API data
Call:
```
ask_page(url, "Describe each filter panel section — what are the exact labels, how many options are shown, and are any labeled generically like 'Filter' or left blank?")
```

Compare:
| Source | Facet count | Labels | Options per facet |
|--------|-------------|--------|-------------------|
| API (direct fetch) | X | Named | N |
| Rendered UI (ask_page) | Y | Actual labels | M |

If X > Y: filters exist in data but aren't rendering → investigate query construction first (is the UI sending `searchType=category`?), then classify as `[FRONTEND INTEGRATION]` or `[STRATEGY]`
If labels differ: facet name resolver is broken → `[FRONTEND INTEGRATION]`
If options differ: API returns more than UI shows → possible pagination or rendering cap

### Step 4: Classify each gap
For every discrepancy, assign:
- Root cause type (see SKILL.md classification table)
- Owning team
- Estimated fix complexity: Low (config), Medium (code), High (architecture)

## Real Example: insight.com Monitors Category

insight.com uses a custom Fusion/Solr-based search API at `/gapi/product-search/search`. It is **not** Algolia, Coveo, or SFCC — the scraper's platform detection will likely report unknown or misidentify it.

**Key API parameters for browse pages:**
| Parameter | Value | Notes |
|-----------|-------|-------|
| `searchType` | `category` | Required for browse mode — fires merchandising rules |
| `category` | `hardware/monitors` | Category code from URL path |
| `salesOrg` | `2400` | Required — omitting returns no results |
| `userSegment` | `CES` | Consumer/SMB segment default |
| `rows` | `24` | Page size |
| `start` | `0` | Pagination offset |
| `lang` | `en_US` | |
| `locale` | `en_US` | |

**Why `searchType=category` matters:** The search pipeline applies different merchandising rules depending on the query type. For browse pages, the frontend must pass `searchType=category` + the `category` code to get the proper facet set and product ranking. A keyword search (`searchType=keyword`) against the same category returns results but without the full facet schema and without browse-mode boosting. This is expected behavior — not a bug.

**Facet bucket schema (3 variants):**
- Category-type facets: `{val, display, count}` — use `display` as the label
- Price/range facets: `{val, count, label, key}` — use `label` as the display name
- Default facets (manufacturer, etc.): `{val, count}` — use `val` as the label

**Full 13-facet inventory for monitors (benchmark):**
| Facet | Options | Notes |
|-------|---------|-------|
| Category | ~8 | Sub-categories (Led Monitors, Curved Monitors, etc.) |
| Price | ~6 | Range buckets |
| Manufacturer | ~20+ | Brand list |
| Screen Size | ~10 | Inch ranges |
| Display Position Adjustments | ~4 | Tilt, pivot, height, swivel |
| Interface | ~6 | HDMI, DisplayPort, USB-C, etc. |
| Native Resolution | ~6 | 1080p, 1440p, 4K, etc. |
| Form Factor | ~3 | Standard, widescreen, ultrawide |
| Audio Output Type | ~2 | Built-in speakers, no audio |
| Product Line | ~5 | Brand sub-lines |
| Display Type | ~3 | IPS, VA, TN |
| Curved Screen | 2 | Yes/No |
| New Product | 2 | Yes/No |

The UI renders only 3 of these 13 facets by default (Category, Price, Manufacturer). This gap is **not** purely a frontend integration issue — it is a deliberate merchandising/UX decision. Investigate before classifying: are the missing facets surfaced elsewhere (e.g., on the PDP), or are they genuinely absent from the browse experience?

**Findings table:**
| Finding | API says | UI shows | Root cause | Owner |
|---------|----------|----------|------------|-------|
| Facet count | 13 facets available | 3 rendered | Query construction required: `searchType=category` + category code must be sent; remaining facets are a merchandising configuration gap | Merchandising / Product |
| Ecommerce tracking | 0 product impressions fired | Products visible on page | GTM ecommerce event not wired to product listing renders | Analytics/Engineering |
| Product title | Internal SKU string in data feed | "DELL LATITUDE 7450 XCTO INTEL BE200 WLAN DRIVER 14" displayed verbatim | Raw feed field used without normalization | Merchandising/catalog |
| Mobile render | Products in DOM | Blank white screen | JS render dependency blocking mobile viewport | Engineering |
| "NEW" badge | `isNew: false` in scraper data layer | Visual "NEW" badge on product card | Feed field not propagating to structured data / GTM layer; badge renders from a separate product attribute | Catalog / Analytics |

## Signals That Indicate Frontend Integration Gaps (not data problems)

- Facets present in scrape response but labeled "Unknown Facet" in UI
- `networkIntel.platforms` detects Algolia/Coveo but facet options are sparse compared to typical search API responses
- `dataLayersDetected` shows GTM present but `ecommerceEvents` array is empty
- `fingerprint.funnelReadiness` = "weak" despite rich product data in the scrape
- Products visible in scrape response but mobile screenshot is blank

## Signals That Indicate Data Quality Problems (not engineering problems)

- Product titles contain internal SKU codes, driver file names, or part numbers
- Prices show inconsistent formatting (some with "Your price:", some with "List price:", some raw)
- Image alt text is identical to the raw title string (no curation)
- Duplicate products with only color/SKU variant differences, shown as separate cards with no grouping
- Description field in scrape response is identical to the title (description field not populated in feed)
