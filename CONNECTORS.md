# Connectors

## Required: merch-connector

This plugin requires the **merch-connector** MCP server to be active in your Cowork session. It powers all storefront scraping, auditing, comparison, and memory capabilities.

| Capability | Tool | Notes |
|------------|------|-------|
| Scrape storefront pages | `scrape_page` | Required for /audit and /compare |
| Run structured audits | `audit_storefront` | Core audit engine |
| Ask questions about a page | `ask_page` | Used to probe specific elements |
| Compare two storefronts | `compare_storefronts` | Required for /compare |
| Recall prior site analysis | `site_memory` | Provides historical context |

## Setup

Ensure the merch-connector is installed and connected before using `/audit` or `/compare`. If you see tool-not-found errors, check that the connector is active in your session's MCP settings.
