# Herdr Engine Auto-Launch

**Status**: Active (via `pai.engine-launcher` plugin)  
**Version**: 0.1.0  
**Herdr**: 0.7.0+

## Overview

The `pai.engine-launcher` Herdr plugin automatically launches PAI engines when you create tabs or workspaces with engine-specific labels.

## Quick Start

### Create a tab and auto-launch an engine

In Herdr:
1. Press `prefix+c` (default: `ctrl+b`, then `c`)
2. Type a label containing an engine keyword: `PNC`, `builder`, `research`, etc.
3. The engine auto-launches in that tab

Or via CLI:
```bash
herdr tab create --label "PNC - new task"
herdr tab create --label "builder session"
herdr tab create --label "research query"
```

### Create a workspace and auto-launch an engine

```bash
herdr workspace create --label "PNC algorithm" --cwd ~/pai-primary
herdr workspace create --label "PNK builder" --cwd ~/pai-opencode-local
```

## Engine Detection Patterns

| Keyword in label | Engine | CLI |
|-----------------|--------|-----|
| `pnc`, `algorithm`, `claude` | Claude Code | `claude` |
| `pnk`, `builder`, `opencode` | OpenCode | `opencode` |
| `pnx`, `cato`, `codex` | Codex CLI | `codex` |
| `png`, `research`, `antigravity`, `agy` | Antigravity | `agy` |
| `forge` | Codex sandbox | `codex exec --sandbox` |

Patterns are **case-insensitive** and match **anywhere** in the label.

## Examples

### Valid labels that trigger auto-launch

- ✅ `PNC` → launches `claude`
- ✅ `algorithm workspace` → launches `claude`
- ✅ `builder-session` → launches `opencode`
- ✅ `PNK OpenCode` → launches `opencode`
- ✅ `research task` → launches `agy`
- ✅ `PNG Antigravity` → launches `agy`
- ✅ `cato verifier` → launches `codex`
- ✅ `FORGE sandbox` → launches `codex exec --sandbox`

### Labels that do NOT trigger auto-launch

- ❌ `main` (no engine keyword)
- ❌ `logs` (no engine keyword)
- ❌ `dev` (no engine keyword)

## How It Works

1. **Event hook**: Listens for `tab.created` and `workspace.created` events
2. **Label extraction**: Reads `label` field from event JSON
3. **Pattern matching**: Scans label for engine keywords (case-insensitive)
4. **Engine launch**: Runs `herdr pane run <pane_id> <engine>` in background
5. **Non-blocking**: Uses `&` to avoid blocking Herdr UI

## Integration with Existing Workflows

This plugin complements:

- **Manual engine launching**: `bun herdr-engine-launch.ts algorithm`
- **Keybinding shortcuts**: `prefix+alt+1` (PNC), `prefix+alt+2` (PNK), etc.
- **MCP auto-sync**: `pai.mcp-sync` plugin still handles MCP profile switching
- **Conductor.ts**: PAI's engine orchestration layer

The plugin is **passive** — it only acts when you explicitly include an engine keyword in the label.

## Configuration

Plugin manifest: `~/.claude/PAI/Tools/herdr-plugins/pai-engine-launcher/herdr-plugin.toml`

### Modify detection patterns

Edit `engine-launcher.sh` and change the case statement:

```bash
case "$LABEL_LOWER" in
  *myengine*)
    ENGINE="my-custom-cli"
    ;;
  # ... existing patterns
esac
```

Then reload:
```bash
# Herdr reloads plugin manifests on link/unlink/enable/disable
# Or restart herdr server: herdr server stop && herdr
```

## Debugging

### Check if the plugin is active

```bash
herdr plugin list
```

Expected output:
```
- pai.engine-launcher (PAI Engine Launcher) enabled [local:...]
```

### View recent plugin logs

```bash
herdr plugin log list --plugin-id pai.engine-launcher --limit 10
```

### Test manually

Create a tab with an engine label and watch for the engine to launch:

```bash
herdr tab create --label "PNC test"
```

Then check the pane:
```bash
herdr pane list
```

You should see `claude` running in the new tab's root pane.

### Common issues

**Issue**: Engine doesn't launch  
**Fix**: Check that `jq` is installed and the engine CLI is in PATH

```bash
which jq
which claude opencode codex agy
```

**Issue**: Plugin shows as disabled  
**Fix**: Enable it

```bash
herdr plugin enable pai.engine-launcher
```

**Issue**: Wrong engine launches  
**Fix**: Check label pattern priority in `engine-launcher.sh` — first match wins

## Management

### Disable temporarily

```bash
herdr plugin disable pai.engine-launcher
```

### Re-enable

```bash
herdr plugin enable pai.engine-launcher
```

### Uninstall

```bash
herdr plugin unlink pai.engine-launcher
```

## Requirements

- Herdr 0.7.0+
- `jq` (JSON parsing)
- PAI engines in PATH: `claude`, `opencode`, `codex`, `agy`
- Linux (plugin uses bash)

## See Also

- [Herdr Plugin API](https://herdr.dev/docs/plugins/)
- [Herdr Event Hooks](https://herdr.dev/docs/socket-api/#event-subscriptions)
- [PAI Conductor](~/.claude/PAI/Tools/Conductor.ts)
- [Manual Engine Launch](~/.claude/PAI/Tools/herdr-engine-launch.ts)
