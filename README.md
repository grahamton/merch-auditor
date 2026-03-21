# merch-auditor

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Cowork Plugin](https://img.shields.io/badge/Cowork-Plugin-purple)](https://cowork.claude.ai)

> A Cowork plugin for expert storefront auditing, powered by [merch-connector](https://github.com/grahamton/merchGent).

Give Claude deep expertise in evaluating e-commerce storefronts — scoring UX, conversion, copywriting, and competitive positioning — and surfacing specific, prioritized recommendations with tagged ownership.

---

## Why merch-auditor?

Manual storefront auditing is time-consuming, inconsistent, and fragmented. Merchandisers spend hours evaluating competitor sites, checking if data appears correctly, spot-checking product pages, and documenting what changed. merch-auditor automates this workflow — it scrapes storefronts, interrogates data layers, analyzes merchandising quality through expert lenses, and produces scored audit reports with prioritized recommendations.

---

## What It Does

merch-auditor wraps the merch-connector MCP with a structured agent workflow. You point it at a URL, it scrapes the storefront, interrogates the data layer, spot-checks PDPs, and returns a scored audit report saved to your workspace.

**Workflow:**
```
URL → Scrape & Extract → Data Layer Interrogation → PDP Spot-Checks → Scored Report
```

It handles both **B2C** and **B2B/Hybrid** storefronts, routing to the appropriate analysis persona automatically:
- B2C → `conversion_architect` (DTC/retail lens)
- B2B or Hybrid → `b2b_auditor` (procurement, contract pricing, catalog depth)

Reports are saved as markdown to `outputs/` and findings are written back to `site_memory` for score trending across sessions.

---

## Personas

merch-auditor automatically selects the right analysis lens based on storefront signals:

| Persona | When Used | Evaluates |
|---------|-----------|-----------|
| **Conversion Architect** | B2C / DTC storefronts | Funnel friction, UX gaps, copywriting, mobile readiness, checkout experience |
| **B2B Auditor** | B2B / Hybrid storefronts | Procurement readiness, spec completeness, contract pricing visibility, bulk order flow |

---

## Commands

| Command | Usage | What it does |
|---------|-------|--------------| 
| `/audit <url>` | `/audit https://example.com` | Full scored audit across 4 dimensions |
| `/compare <url-1> <url-2>` | `/compare https://a.com https://b.com` | Side-by-side scorecard comparison |
| `/research <url>` | `/research https://example.com [focus]` | Open-ended dossier — competitive research, client prep |
| `/track <url>` | `/track https://example.com` | Change-detection vs. prior audit baseline |

---

## Scoring System

Each audit scores four dimensions on a 1–10 scale (40 points total):

| Score | Grade | Meaning |
|-------|-------|---------|
| 35–40 | A | Exceptional |
| 28–34 | B | Solid |
| 20–27 | C | Needs work |
| < 20 | D | Significant overhaul needed |

Every finding is tagged with its owning team:
- `[DATA QUALITY]` — bad feed data → Merchandising/catalog
- `[FRONTEND INTEGRATION]` — data in API but not rendered → Engineering
- `[UX DESIGN]` — data renders but experience is poor → Design
- `[STRATEGY]` — missing entirely → Product/marketing

---

## Installation

### Requirements
- [Cowork](https://claude.ai/download) desktop app
- **merch-connector ≥ 1.8.0** — installed automatically via `npx` when the plugin runs

### Install the plugin
1. Download or clone this repo
2. Open Cowork → Plugin settings → **Upload plugin folder**
3. Select the repo root (the folder containing `.claude-plugin/`)
4. The plugin activates immediately — no restart needed

### Verify the connection
After installing, run:
```
/audit https://example.com
```
If merch-connector isn't installed, `npx` will pull it automatically on first run.

---

## Output

Audit reports are saved to your selected workspace folder:

```
outputs/audit-[domain]-[YYYY-MM-DD].md
outputs/compare-[domain-a]-vs-[domain-b]-[YYYY-MM-DD].md
outputs/research-[domain]-[YYYY-MM-DD].md
outputs/track-[domain]-[YYYY-MM-DD].md
```

Each run also writes a `site_memory` note via merch-connector, enabling score trend tracking across sessions.

---

## Plugin Structure

```
.claude-plugin/
  plugin.json          ← manifest: version, commands, MCP config
commands/
  audit.md             ← /audit workflow
  compare.md           ← /compare workflow
  research.md          ← /research workflow
  track.md             ← /track workflow
skills/
  storefront-auditing/
    SKILL.md           ← audit framework, persona routing, scoring
    references/
      audit-rubrics.md       ← detailed dimension rubrics
      dual-lens-pattern.md   ← API vs UI gap analysis pattern
CONNECTORS.md          ← MCP connection details
CHANGELOG.md           ← version history
```

---

## Compatibility

| Plugin version | merch-connector |
|----------------|-----------------|
| v0.4.x | ≥ 1.8.0 |
| v0.3.x | ≥ 1.6.0 |

The plugin calls merch-connector via `npx merch-connector@latest` by default. To pin a version, edit `.claude-plugin/plugin.json`:
```json
"args": ["-y", "merch-connector@1.8.0"]
```

---

## Related

- **[merch-connector](https://github.com/grahamton/merchGent)** — the MCP server this plugin wraps. 13 tools, Puppeteer-based scraping, 5 analysis personas.
- Bug reports and roadmap tracked in the [Merch Project Hub](https://www.notion.so/329dee0d562d818a95d7fd5759f0add4) on Notion.

---

## Author

Built by [@grahamton](https://github.com/grahamton).