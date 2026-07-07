# PAI Release Notes

Track system changes that affect multiple engines or add new PAI capabilities.

## 2026-05-24 — MCP Profile System v1.0

**Components added:**

| Component | Path | Type |
|-----------|------|------|
| McpProfiles.ts | `PAI/Tools/McpProfiles.ts` | CLI tool (TypeScript) |
| pai-mcp-sync.sh | `PAI/Tools/pai-mcp-sync.sh` | Tmux hook script (bash) |
| pai-mcp-enable.sh | `PAI/Tools/pai-mcp-enable.sh` | Setup script (bash) |
| pai-tmux-tokens.sh | `PAI/Tools/pai-tmux-tokens.sh` | Token tracker (bash) |
| McpProfiles skill | `skills/McpProfiles/SKILL.md` | Skill documentation |
| Registry config | `~/.config/pai/mcp-registry.json` | 13 MCP server definitions |
| Profile config | `~/.config/pai/mcp-profiles.json` | 9 named profiles |
| Herdr mapping | `~/.config/pai/herdr-mcp-mapping.json` | 11 pattern→profile rules |

**System changes:**

- `~/.tmux.conf`: Added pane-focus-in hook at line 32, token tracker in status-right
- `~/.config/herdr/config.toml`: Added space-switch hook at line 6
- `~/.claude/settings.json`: Added `/mcp` slash command, managed MCP servers section
- `PAI/Tools/Conductor.ts`: Added `syncMcpProfile()` call in engine dispatch (lines 53-63)
- `skills/skill-index.json`: Registered McpProfiles skill

**Integration points:**
- Pulse notification endpoint `http://localhost:31337/notify` on profile switch
- Voice config from `MEMORY/STATE/voice-config.json`
- Token data from `MEMORY/STATE.claude/token-budget.jsonl` and `usage-canonical-cache.json`
- Conductor dispatch for PNC/PNG/PNX/PNK profile pre-sync

**No migration needed.** Config files auto-create on first CLI invocation. Hooks installed via `pai-mcp-enable.sh`.
