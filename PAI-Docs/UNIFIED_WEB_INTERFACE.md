# PAI Unified Web Interface

**Single URL for all PAI engines and tools:**
```
https://pai-primary.taild73141.ts.net/
```

## Quick Access

### Navigation (Top Bar)
- **💬 Chat** (Alt+1) - OpenCode conversation
- **🗂️ Workspaces** (Alt+2) - All sessions across engines
- **🔊 Voice** (Alt+3) - Voice player controls

### Widgets (Bottom-Right)
- **💻 Terminal** (purple button or Ctrl+`) - Embedded tmux terminal
- **🎤 Microphone** (blue button or Ctrl+Shift+M) - Voice input (STT)
- **🔊 Voice Status** (green/gray) - Voice output indicator (TTS)

## Workspaces View

**URL:** `/workspaces`

Shows all sessions grouped by engine in a card grid:
- **PNK** (OpenCode) - Purple badge
- **PNC** (Claude Code) - Blue badge
- **PNX** (Codex) - Green badge
- **PNG** (Antigravity) - Red badge

### Opening Sessions

**OpenCode (PNK):**
1. Click "Open" button
2. Session opens in current window
3. Continue where you left off

**Terminal Engines (PNC/PNX/PNG):**
1. Click "Open" button
2. Command copied to clipboard
3. Open Terminal widget (💻 or Ctrl+`)
4. Paste and run command
5. Session resumes in terminal

## Services

All services auto-start on boot via systemd:

```bash
# OpenCode web GUI (port 8123)
systemctl --user status opencode-web.service

# PAI Pulse daemon (port 31337)
systemctl --user status pai-pulse.service

# Terminal web interface (port 7681)
systemctl --user status herdr-terminal-web.service
```

## Architecture

**OpenCode Web** → Tailscale HTTPS → **Pulse API** → Session Registry
                                     ↓
                                Terminal (ttyd)

**Components:**
- **OpenCode** - Web GUI for PNK sessions
- **Pulse** - Unified daemon (voice, sessions API, cockpit)
- **Herdr** - tmux session management (PNC/PNX/PNG)
- **ttyd** - Terminal web interface
- **Tailscale** - Secure HTTPS proxy

## Data Sources

**Session Registry** (`~/.claude/PAI/Tools/SessionRegistry.ts`):
- **PNK** - OpenCode SQLite DB (`~/.local/share/opencode/opencode.db`)
- **PNC** - Claude Code session-names.json
- **PNX/PNG** - Herdr workspace API (future)

**APIs:**
- `GET /sessions` - JSON session list
- `GET /workspaces` - Workspaces UI
- `GET /sessions/ui` - Detailed sessions table
- `GET /voice/player` - Voice player interface

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| Alt+1 | Chat (OpenCode) |
| Alt+2 | Workspaces |
| Alt+3 | Voice Player |
| Ctrl+` | Toggle Terminal |
| Ctrl+Shift+M | Voice Input (STT) |

## Voice System

**Status:** Disabled per user preference (audio routing unclear)

**When enabled:**
- TTS plays automatically in browser
- STT activates via microphone button
- Routing: Pulse → Legion Voicebox → Kokoro TTS

**To enable:** Update voice preferences in PAI settings

## Troubleshooting

**Can't access web interface:**
```bash
# Check OpenCode web service
systemctl --user status opencode-web.service

# Check Pulse daemon
systemctl --user status pai-pulse.service

# Check Tailscale
tailscale status
```

**Sessions not showing:**
```bash
# Rebuild session registry
bun ~/.claude/PAI/Tools/SessionRegistry.ts build

# Check API
curl http://localhost:31337/sessions
```

**Terminal not working:**
```bash
# Check ttyd service
systemctl --user status herdr-terminal-web.service

# Restart if needed
systemctl --user restart herdr-terminal-web.service
```

## URLs Reference

| Interface | URL |
|-----------|-----|
| Chat | https://pai-primary.taild73141.ts.net/ |
| Workspaces | https://pai-primary.taild73141.ts.net/workspaces |
| Sessions List | https://pai-primary.taild73141.ts.net/sessions/ui |
| Voice Player | https://pai-primary.taild73141.ts.net/voice/player |
| Terminal | https://pai-primary.taild73141.ts.net/terminal |
| Cockpit | https://pai-primary.taild73141.ts.net/cockpit |
| Health | https://pai-primary.taild73141.ts.net/api/pulse/health |

---

**Last Updated:** 2026-07-07  
**Version:** PAI 5.1.0 Unified Web Interface
