# Changelog

## v0.4.0 — 2026-03-21
- Added `/research` and `/track` commands
- Fixed relative `outputs/` path bug (PLUGIN-003)
- Established `plugin-source/` as single source of truth for editable plugin files
- Added MCP compatibility requirement (`merch-connector >=1.8.0`)

## v0.3.0
- Added B2B persona routing (`b2b_auditor` for Hybrid/B2B sites)
- Dual-lens audit pattern (API data layer vs. UI rendering)
- PDP spot-check procedure added to skill
- Output persistence requirements added to all commands

## v0.2.0
- `/compare` command added
- Scoring system formalized (40-point, 4-dimension)
- Finding tags introduced: `[DATA QUALITY]`, `[FRONTEND INTEGRATION]`, `[UX DESIGN]`, `[STRATEGY]`

## v0.1.0
- Initial release: `/audit` command, `storefront-auditing` skill
