---
title: "Per-Session MCP Profile Manager"
type: idea
tags: [mcp, profiles, herdr, session-management, cross-engine]
created: 2026-05-24
updated: 2026-05-24
quality: 5
source_session: mcp-profile-system-v1
related: []
---

# Per-Session MCP Profile Manager

## Thesis
A PAI-native per-session MCP profile system eliminates the waste of loading all MCP servers in every session. By mapping tmux session names (Herdr spaces) to MCP profiles, the right servers load automatically based on what you're doing — research gets web-search + qdrant, development gets github + filesystem, infrastructure gets nothing heavy.

## Evidence

Built across multiple sessions in two days (2026-05-24). Key decisions:

- **Adopted PAI-native over Agent-Deck** — Agent-Deck's Chrome extension approach was heavier than needed. PAI already had `settings.json` patching, tmux hooks, and Pulse notifications. The profile system uses those primitives directly.
- **Three config files** pattern: `mcp-registry.json` (canonical server definitions), `mcp-profiles.json` (named compositions), `herdr-mcp-mapping.json` (session→profile rules). Separates concerns cleanly.
- **Two sync triggers** for reliability: tmux pane-focus-in hook (every pane switch) + Herdr space-switch hook (every workspace change). The pane-focus hook is ~30ms on cache hits.
- **Conductor.ts integration** — engine dispatcher calls `syncMcpProfile()` before launching any sub-engine (PNG→research, PNX→development, PNK→minimal). Fire-and-forget subprocess so MCP failures never block dispatch.
- **9 profiles** covering: default, research, development, minimal, infrastructure, writing, security, full, browser-debug — deployed across 11 Herdr session patterns including wildcard fallback.
- **Plugin-type MCP support** — the registry supports both stdio (`command`+`args`) and HTTP (`type: "http"`+`url`) server definitions, needed for GitHub Copilot MCP and browser-devtools.
- **Token budget in tmux** — `pai-tmux-tokens.sh` reads `token-budget.jsonl` and `usage-canonical-cache.json` for live budget display in the status bar.
- **Combined dashboard** — `pai-dashboard.sh` shows MCP profile + token budget + Pulse handoffs + system state in a single TUI with `--once`, `--json`, and live-watch modes.
- **Pulse notifications** — profile switches fire-and-forget to Chatterbox TTS via the pai-pulse proxy on port 31337.

## Implications

- **MCP profiles should be the default model** — shipping the full 13-server registry in every Claude Code session is wasteful. The per-session approach saves ~2-5s of startup time and reduces token overhead from unnecessary server heartbeats.
- **The tmux-based session classifier is the right abstraction** — tmux session names encode intent naturally (research/, dev/, infra/). No ML needed. Pattern matching with regex wildcards handles the long tail.
- **Cross-engine MCP profiles are a solved problem** — the `McpProfiles.ts` CLI patches Claude Code, OpenCode, and Codex configs with `--all`. The `syncMcpProfile()` call in Conductor ensures every engine starts with the right profile for its task.
- **Future work could add MCP server health checks** — probe each server on profile switch to surface connectivity issues before the engine depends on them. The dashboard already has the framework for this.
- **The three-config-file pattern is reusable** — any PAI subsystem that needs session-scoped configuration (tools, environment variables, API keys) could follow the same registry/profile/mapping split.
