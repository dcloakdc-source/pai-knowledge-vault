---
title: "Daemon MCP Server - Cloudflare Pages Function"
type: idea
tags: [daemon, mcp, cloudflare-pages, json-rpc, personal-website]
created: 2026-05-25
updated: 2026-05-25
quality: 6
source_session: 20260525-020000_daemon-deploy-personalization
related:
  - slug: mcp-profile-system
    type: part-of
  - slug: daemon-deployment
    type: supports
  - slug: cloudflare-pages-domain-gotcha
    type: extends
---

# Daemon MCP Server

## Thesis
A daemon site (personal AI profile) can serve as both a static showcase and a live MCP endpoint by adding a Cloudflare Pages Function at `/mcp` that parses a simplified INI-format `daemon.md` and exposes its sections as JSON-RPC 2.0 tools.

## Evidence
- Built `functions/mcp.ts` — handles `tools/list` (13 tools) and `tools/call` (section extraction) via JSON-RPC
- `daemon.md` uses `[SECTION_NAME]` headers (not YAML) that the function parses with a simple line-by-line scanner
- Deployed alongside static Astro build — zero additional infrastructure, bundled automatically by `wrangler pages deploy`
- Dashboard fetches from `/mcp` with relative paths — works on preview deployments and custom domains
- Built with zero dependencies (Bun-native `Bun.serve` not required for the function, just standard `fetch`)

## Implications
- Pages Functions are the simplest path to add live API to a static site — no separate worker deployment needed
- The `daemon.md` format (plain `[SECTION]` headers) is simpler than YAML frontmatter and works for both static consumption and parsing
- For future: add streaming/SSE transport for full MCP compliance, not just JSON-RPC over HTTP POST
- Personalization required updating 16 Daniel Miessler references across 5 source files — fork management is essential for open-source frameworks
