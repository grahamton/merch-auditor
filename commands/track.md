---
description: Show what changed on a storefront since the last audit
allowed-tools: mcp__merch-connector__scrape_page, mcp__merch-connector__ask_page, mcp__merch-connector__site_memory, Write, Bash
argument-hint: <storefront-url>
---

Run a change-detection pass on $ARGUMENTS and compare against the last audit in site_memory.

Use this to monitor a storefront over time — catch price changes, new products, removed facets, copy updates, or regressions in performance without running a full scored audit.

## Step 1 — Load prior baseline

Call `site_memory(action="read", url=$ARGUMENTS)`.

Extract:
- Prior audit scores (if any)
- Prior product snapshot (titles, prices)
- Prior facet list
- Prior performance numbers (FCP, load time)
- Date of last scrape

If no prior data exists, tell the user this is a first visit and run `/audit` instead.

## Step 2 — Scrape current state

Call `scrape_page(url=$ARGUMENTS, include_screenshot=true, mobile_screenshot=true, max_products=20)`.

The scrape response includes a `changes` object — use it as the primary source for deltas.

## Step 3 — Produce a change report

Structure the output as:

---
## Change Report: $ARGUMENTS
**Compared:** [last scrape date] → [today]

### Products
- New products: [list titles]
- Removed products: [list titles]
- Price changes: [product → was $X, now $Y]
- No changes: [count unchanged]

### Filters & Discovery
- Facets added: [list]
- Facets removed: [list]
- Sort options changed: [yes/no, details]

### Performance
| Metric | Last | Now | Delta |
|--------|------|-----|-------|
| FCP | Xms | Yms | +/-Z |
| Load time | Xms | Yms | +/-Z |
| Resources | X | Y | +/-Z |

### Mobile Rendering
[Still blank / Fixed / Still rendering / Newly broken]

### Score Trend (if prior audit exists)
| Dimension | Last | Change |
|-----------|------|--------|
| UX & Design | X/10 | — |
| Conversion | X/10 | — |
| Copywriting | X/10 | — |
| Positioning | X/10 | — |
| **Total** | X/40 | — |

### Notable Changes
[Anything that warrants attention — new promotions, copy changes spotted via ask_page, new trust signals, missing elements]

### Recommended Next Action
- If significant changes detected → suggest running `/audit` for a full scored re-evaluation
- If minor/no changes → confirm site is stable, note when next check makes sense
---

## Step 4 — Update site_memory

Call `site_memory(action="write", url=$ARGUMENTS, note="Track [date]: [1-sentence summary of what changed, or 'No significant changes detected'].")`.

## Step 5 — Save the change report

Use Bash to resolve the workspace path and create the outputs directory:
```bash
WS=$(find /sessions/*/mnt -maxdepth 1 -type d -name "Merch-connector" ! -path "*local-plugins*" 2>/dev/null | head -1) && mkdir -p "$WS/outputs" && echo "$WS"
```

Get today's date: `date +%Y-%m-%d`

Write the change report to `$WS/outputs/track-[domain]-[YYYY-MM-DD].md`.
