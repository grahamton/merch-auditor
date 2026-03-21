# Merch Project Hub

This workspace is the **central hub for both projects** in the merch ecosystem:

| Project | Repo | What it is |
|---------|------|------------|
| **merch-connector** (MCP) | [github.com/grahamton/merchGent](https://github.com/grahamton/merchGent) · `C:\dev\merchGent` | The MCP server. npm package: `merch-connector`. 13 tools, Puppeteer-based scraping, 5 analysis personas, multi-AI provider support. |
| **merch-auditor** (plugin) | [github.com/grahamton/merch-auditor](https://github.com/grahamton/merch-auditor) · `C:\dev\merch-auditor` | The Cowork plugin. 4 commands, storefront-auditing skill. Wraps the MCP with agent workflows and UX. |

Work done here covers: testing end-to-end workflows, finding and triaging bugs, designing improvements, and building the roadmap for both projects.

---

## Two Projects, Two Bug Queues

**When a bug is found, classify it before filing:**

| Symptom | Owner | Fix goes in |
|---------|-------|-------------|
| Tool returns wrong/missing data | MCP | merchGent repo |
| Tool crashes or throws | MCP | merchGent repo |
| Agent skips a step / wrong order | Plugin | merch-auditor plugin |
| Skill loads wrong rubric / wrong persona | Plugin | skills/storefront-auditing/ |
| Score seems wrong but data is right | Plugin | audit-rubrics.md |
| Output file not saved / wrong filename | Plugin | command or agent file |
| site_memory not written after run | Plugin | command or agent file |

File bugs in `bugs/[YYYY-MM-DD]-[short-description].md` with: tool/command used, URL, expected behavior, actual behavior, and log output from `get_logs`.

---

## MCP: merch-connector (merchGent)

**npm:** `merch-connector` → distributed via `npx merch-connector`
**Current version:** 1.8.0 (25+ shipped features)
**Node:** 18+

**13 tools:**

| Tool | Purpose |
|------|---------|
| `scrape_page` | Puppeteer scrape — products, facets, screenshots, network intel, data layer |
| `scrape_pdp` | Scrapes a product detail page |
| `get_category_sample` | Samples products from a category |
| `ask_page` | Natural-language question about current scraped page |
| `interact_with_page` | Clicks, scrolls, interacts with page elements |
| `audit_storefront` | Structured built-in audit engine |
| `compare_storefronts` | Head-to-head comparison of two URLs |
| `merch_roundtable` | 5-persona streaming debate analysis |
| `site_memory` | Read/write persistent per-domain notes across sessions |
| `clear_session` | Resets scraper session state |
| `get_logs` | Retrieves connector logs for debugging |
| `save_eval` / `list_evals` | Manage evaluation test cases |

**5 personas:** Floor Walker, Auditor, Scout, B2B Auditor, Conversion Architect

**Server files:** `analyzer.js`, `eval-store.js`, `index.js`, `network-intel.js`, `scraper.js`, `site-memory.js`

**Data persists to:** `~/.merch-connector/` by default

**Strategic direction:** Pivoting to decouple analysis from scraping — Firecrawl (or similar) handles reliable data acquisition, merch-connector focuses on intelligent analysis. Roadmap includes `analyze_products` tool accepting JSON from any source.

---

## Plugin: merch-auditor

**Version:** 0.3.0 | **MCP config:** references `merch-connector` via npx

**4 commands:**

| Command | Usage | What it does |
|---------|-------|--------------|
| `/audit <url>` | `/audit https://example.com` | Full scored audit across 4 dimensions, saves report + site_memory |
| `/compare <url-1> <url-2>` | `/compare https://a.com https://b.com` | Side-by-side comparison with scorecard |
| `/research <url>` | `/research https://example.com [focus area]` | Open-ended dossier — competitive research, client prep |
| `/track <url>` | `/track https://example.com` | Change-detection pass vs. prior audit baseline |

---

## Audit Scoring

Each audit scores 4 dimensions (1–10 each), totaling 40 points:

| Score | Grade | Meaning |
|-------|-------|---------|
| 35–40 | A | Exceptional |
| 28–34 | B | Solid |
| 20–27 | C | Needs work |
| < 20 | D | Significant overhaul needed |

**Finding tags — every issue tagged with its owner:**
- `[DATA QUALITY]` — bad feed data → Merchandising/catalog team
- `[FRONTEND INTEGRATION]` — data in API but not rendered → Engineering
- `[UX DESIGN]` — data renders but experience is poor → Design/content
- `[STRATEGY]` — missing entirely → Product/marketing

---

## Output Conventions

All saved outputs go in `outputs/` — naming pattern:
- `outputs/audit-[domain]-[YYYY-MM-DD].md`
- `outputs/track-[domain]-[YYYY-MM-DD].md`
- `outputs/research-[domain]-[YYYY-MM-DD].md`
- `outputs/compare-[domain-a]-vs-[domain-b]-[YYYY-MM-DD].md`

After every run, write a `site_memory` note for score trends and cross-session context.

---

## Plugin Development Workflow

### Two locations, one source of truth

| Location | Role | Who edits it |
|----------|------|--------------|
| `C:\dev\merch-auditor` | **Canonical source of truth.** Git repo lives here. Commits and releases happen here. | You, manually |
| `plugin-source/` (this workspace) | **Working mirror.** I read and edit files here during testing sessions. Reinstall Cowork from here. | Me (AI), during sessions |
| `.local-plugins/` | Installed copy. Read-only — never edit directly. | Cowork, on install |

**The sync direction is always one-way:**
```
plugin-source/ (I edit here)
       ↓  you copy changed files
C:\dev\merch-auditor (you commit here)
       ↓  git push
github.com/grahamton/merch-auditor
```

I cannot touch `C:\dev\merch-auditor` — it's outside this workspace. After a session where I've made changes to `plugin-source/`, copy the changed files to your dev folder and commit.

**To make changes to the plugin:**
1. I edit files in `plugin-source/` during the session
2. You copy changes to `C:\dev\merch-auditor` and commit
3. Reinstall: open Cowork plugin settings → remove current merch-auditor → upload `plugin-source/` folder (or `C:\dev\merch-auditor` directly)
4. The new version becomes active immediately

**Source structure:**
```
plugin-source/          ← mirrors C:\dev\merch-auditor exactly
├── .claude-plugin/
│   └── plugin.json     ← manifest (version, commands, mcpServers, compatibility)
├── commands/           ← slash command files (/audit, /compare, /research, /track)
├── skills/
│   └── storefront-auditing/
│       ├── SKILL.md
│       └── references/
│           ├── audit-rubrics.md
│           └── dual-lens-pattern.md
├── CHANGELOG.md
├── CONNECTORS.md
├── README.md
└── .gitignore
```

Never edit files in `.local-plugins/` directly — changes there don't persist and won't survive a reinstall.

---

## Notion Project Hub

All bug tracking, roadmap, and audit archive live in Notion:

| Database | URL |
|----------|-----|
| 🛍️ Merch Project Hub | https://www.notion.so/329dee0d562d818a95d7fd5759f0add4 |
| 🐛 Bug Tracker | https://www.notion.so/3bab2cc83bf84b9e8004e04e2503f09f |
| 🗺️ Roadmap | https://www.notion.so/1c50e09eff6b463d95890a6b800e9c40 |
| 📊 Audit Archive | https://www.notion.so/ac5442bce2a54b70a2330a73a11254c5 |

When a new bug is found: file in `bugs/` folder AND add a row to the Bug Tracker in Notion.
When an audit run completes: add a row to the Audit Archive in Notion.

---

## Key Docs in This Workspace

| File | Purpose |
|------|---------|
| `merch-unification-plan.md` | Architecture plan for cohesive plugin + MCP integration |
| `bugs/` | Bug reports, classified by plugin vs. MCP |
| `outputs/` | Audit run archive |
| `plugin-source/` | Editable plugin source — make changes here, then reinstall |
| `skills/storefront-auditing/references/audit-rubrics.md` | Detailed scoring rubrics |
| `skills/storefront-auditing/references/dual-lens-pattern.md` | API vs UI gap analysis pattern |

---

## Skill Reference

The `merch-auditor:storefront-auditing` skill contains:
- Four-dimension audit framework and scoring rubrics
- Persona routing (B2C → `conversion_architect`, B2B → `b2b_auditor`)
- Dual-lens audit pattern (API data layer vs. UI rendering)
- PDP spot-check procedure
- Output persistence requirements
