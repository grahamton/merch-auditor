---
description: Show what changed on a storefront since the last check
allowed-tools: mcp__merch-connector__acquire, mcp__merch-connector__site_memory, Write, Bash
argument-hint: <storefront-url>
---

Run a change-detection pass on $ARGUMENTS and compare against the baseline in site_memory.

Use this to monitor a storefront over time — catch price changes, new products, removed facets, or performance regressions without running a full audit.

## Step 1 — Load prior baseline

Call `site_memory(action="read", url=$ARGUMENTS)`.

Extract:
- Prior product snapshot (titles, prices, count)
- Prior facet list and sort options
- Prior performance numbers (FCP, loadComplete)
- Prior analytics platforms
- Date of last scrape
- Any custom notes or trend data

If no prior data exists, tell the user this is a first visit and suggest running `/research` or `/audit` instead.

## Step 2 — Acquire current state

Call `acquire(url=$ARGUMENTS, pdp_sample=0)`.

Note: `pdp_sample=0` for speed — change detection doesn't need PDP drill-down. Payload includes all you need for delta detection.

Payload includes:
- `products[]` — current catalog
- `facets[]` — current filters
- `sort` — current sort options
- `performance` — FCP, loadComplete, resourceCount
- `commerce.mode`, `trustSignals`
- **`changes`** — if prior baseline exists, comprehensive delta object

## Step 3 — Produce a change report

Structure as:

---
## Change Report: [Domain]
**Compared:** [last scrape date] → [today]

### Products
- New products: [count + sample titles if >3]
- Removed products: [count + sample titles if >3]
- Price changes: [product title: was $X → now $Y] (list only >1% variance)
- Unchanged: [count]

### Facets & Sort
- Facets added: [list from `payload.changes.facetChanges.added`]
- Facets removed: [list from `payload.changes.facetChanges.removed`]
- Sort options changed: [yes/no, detail if yes]

### Performance
| Metric | Last | Now | Delta |
|--------|------|-----|-------|
| FCP (ms) | X | Y | +/- |
| Load Complete (ms) | X | Y | +/- |
| Resources | X | Y | +/- |

### Commerce Mode
- B2C/B2B/Hybrid: [from `payload.commerce.mode`]
- Price transparency: [from `payload.commerce.priceTransparency`]

### Trust Signals
[Summary changes: badges, free shipping messaging, return policy visibility from `payload.trustSignals`]

### Recommended Next Action
- **If changes > threshold** (5+ new/removed products, any price delta >15%, facets added/removed, FCP +1000ms) → suggest `/audit` for full re-evaluation
- **If minor/no changes** → confirm site is stable, suggest when next track makes sense (e.g., "Check again in 7 days")

---

## Step 4 — Update site_memory

Call `site_memory(action="write", url=$ARGUMENTS, note="Track [date]: [1-sentence summary of what changed, or 'No significant changes detected'].")`.

## Step 5 — Save the change report

Use Bash to resolve the workspace path:
```bash
WS=$(find /sessions/*/mnt -maxdepth 1 -type d -name "Merch-connector" ! -path "*local-plugins*" 2>/dev/null | head -1) && mkdir -p "$WS/outputs" && echo "$WS"
```

Get today's date: `date +%Y-%m-%d`

Write the change report to `$WS/outputs/track-[domain]-[YYYY-MM-DD].md`.
