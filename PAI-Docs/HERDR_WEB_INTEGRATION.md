# Herdr Web Integration

Herdr now supports browser-based sessions alongside terminal sessions, allowing you to launch and manage OpenCode web sessions through the same Herdr interface.

## Overview

**Traditional Herdr** manages terminal-based tmux panes for Claude, OpenCode, and Codex.

**Herdr Web** extends this to browser-based sessions, enabling:
- Launch OpenCode in browser instead of terminal
- Track web sessions in Herdr registry
- Attach to existing web sessions by ID
- Unified view of both terminal and web sessions

## Quick Start

### Launch New Web Session

```bash
# Long form
bun ~/.claude/PAI/Tools/HerdrWebLauncher.ts launch opencode "My Session"

# With alias (after sourcing ~/.bashrc)
hweb launch opencode "My Session"
```

This will:
1. Create a new web session entry
2. Generate a unique session ID
3. Open https://pai-primary.taild73141.ts.net/ in your default browser
4. Track the session in `~/.claude/MEMORY/STATE/herdr-web-sessions.json`

### Attach to Existing Session

```bash
# Attach by web session ID
hweb attach web_1783292862627_wfs8g9yn

# Attach by OpenCode session ID
hweb attach ses_abc123

# Attach by direct URL
hweb attach "https://pai-primary.taild73141.ts.net/?session=ses_abc123"
```

### List All Web Sessions

```bash
hweb list
```

Output:
```
═══════════════════════════════════════════════════════════════
HERDR WEB SESSIONS
Last updated: 7/5/2026, 11:07:42 PM
═══════════════════════════════════════════════════════════════

ID                    ENGINE    STATUS      TITLE
───────────────────────────────────────────────────────────────
web_1783292862627_wf  opencode  active      Test Web Session

Total: 1 web sessions
═══════════════════════════════════════════════════════════════
```

### Get Session URL

```bash
hweb url web_1783292862627_wfs8g9yn
# Output: https://pai-primary.taild73141.ts.net/?session=ses_abc123
```

## Commands Reference

| Command | Description | Example |
|---------|-------------|---------|
| `launch <engine> [title]` | Launch new web session | `hweb launch opencode "Debug"` |
| `attach <session-id>` | Open existing session in browser | `hweb attach ses_abc123` |
| `list` | Show all web sessions | `hweb list` |
| `url <session-id>` | Get session URL | `hweb url web_123` |

## Supported Engines

Currently supported:
- ✅ **opencode** — OpenCode web interface at https://pai-primary.taild73141.ts.net/

Planned:
- 🔜 **claude** — Claude Code web interface (when available)
- 🔜 **codex** — Codex web interface (when available)

## Integration with HerdrCockpit

HerdrCockpit now recognizes `opencode-web` as an engine type:

```bash
hc launch council-web   # Launch council with web-based panes (future)
```

The `ENGINES` map in HerdrCockpit.ts includes:

```typescript
const ENGINES: Record<string, string[]> = {
  claude: ["claude"],
  codex: ["codex"],
  opencode: ["opencode"],
  "opencode-web": ["bun", "/home/duane/.claude/PAI/Tools/HerdrWebLauncher.ts", "launch", "opencode"],
};
```

## Session Registry

Web sessions are stored in:
```
~/.claude/MEMORY/STATE/herdr-web-sessions.json
```

Format:
```json
{
  "sessions": [
    {
      "id": "web_1783292862627_wfs8g9yn",
      "engine": "opencode",
      "session_id": "ses_abc123",
      "url": "https://pai-primary.taild73141.ts.net/?session=ses_abc123",
      "title": "My Debug Session",
      "created_at": "2026-07-05T23:07:42.627Z",
      "last_accessed": "2026-07-05T23:10:00.000Z",
      "status": "active"
    }
  ],
  "last_updated": "2026-07-05T23:07:42.630Z"
}
```

## Browser Detection

The launcher automatically detects your platform and uses the appropriate command:

- **Linux**: `xdg-open`
- **macOS**: `open`
- **Windows**: `start`

If browser opening fails, the URL is printed to console for manual opening.

## Unified Session View (Future)

Planned integration with SessionRegistry.ts to show both terminal and web sessions in a unified view:

```bash
/sessions --include-web
```

Output would show:
```
ENGINE  TYPE        STATUS      TITLE
──────────────────────────────────────────
PNK     terminal    active      Terminal session
PNK     web         active      Web session
PNC     terminal    background  Claude Code session
```

## Workflows

### Debug Session in Browser

```bash
# Launch web session
hweb launch opencode "Debug API error"

# Browser opens automatically
# Work in the web UI
# Session is tracked with title "Debug API error"

# Later, reattach
hweb list
hweb attach web_1783292862627_wfs8g9yn
```

### Switch from Terminal to Web

```bash
# Currently in terminal OpenCode
# Want to continue in browser

# Get current session ID from terminal
# (visible in OpenCode TUI or via `opencode session list`)

# Open in browser
hweb attach ses_abc123

# Browser opens with the same session
# Continue work in browser instead of terminal
```

### Share Session with Another Device

```bash
# On pai-primary
hweb url web_1783292862627_wfs8g9yn
# Output: https://pai-primary.taild73141.ts.net/?session=ses_abc123

# Copy URL
# On laptop/tablet/phone (connected to Tailscale)
# Open URL in browser
# Same session, different device
```

## Shell Aliases

Add to `~/.bashrc` (already added):

```bash
# Herdr Web Launcher
alias hweb='bun ~/.claude/PAI/Tools/HerdrWebLauncher.ts'
```

Reload:
```bash
source ~/.bashrc
```

## Troubleshooting

**Browser doesn't open:**
```bash
# Check if xdg-open is available
which xdg-open

# Manually open the URL shown in output
```

**Session not found:**
```bash
# List all sessions to verify ID
hweb list

# Check registry file
cat ~/.claude/MEMORY/STATE/herdr-web-sessions.json
```

**Wrong URL:**
```bash
# Web base URL is hardcoded in HerdrWebLauncher.ts
# Edit if you're using a different domain
const WEB_BASE_URL = "https://pai-primary.taild73141.ts.net";
```

## Files

| File | Purpose |
|------|---------|
| `~/.claude/PAI/Tools/HerdrWebLauncher.ts` | Web session launcher/manager |
| `~/.claude/MEMORY/STATE/herdr-web-sessions.json` | Web session registry |
| `~/.claude/PAI/Tools/HerdrCockpit.ts` | Terminal session manager (extended) |

## Future Enhancements

- [ ] Real-time session status via WebSocket
- [ ] Embed web sessions in terminal via tmux splits
- [ ] Web → Terminal handoff (continue browser session in terminal)
- [ ] Multi-browser support (Firefox, Chrome, Safari profiles)
- [ ] Session snapshots/bookmarks
- [ ] Team session sharing
