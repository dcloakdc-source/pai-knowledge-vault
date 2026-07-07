# PAI Session Registry

Unified session tracking across all PAI engines with human-readable identifiers.

## Overview

The Session Registry provides a single view of all active, background, and idle sessions across:
- **PNC** (Claude Code)
- **PNK** (OpenCode)
- **PNX** (Codex CLI)
- **PNG** (Antigravity/Gemini CLI)
- **PNO** (Ollama local)

## Features

✅ Human-readable session titles  
✅ Session status tracking (active/background/idle/blocked/done)  
✅ Engine identification  
✅ Web GUI integration  
✅ TUI `/sessions` command  
✅ HTTP API endpoint  
✅ Herdr pane integration  
✅ Session URLs for web access

## Usage

### TUI Command (OpenCode)

```bash
/sessions
```

Shows formatted table of all sessions in the OpenCode TUI.

### CLI Tool

```bash
# Build/refresh registry
bun ~/.claude/PAI/Tools/SessionRegistry.ts build

# List sessions (formatted)
bun ~/.claude/PAI/Tools/SessionRegistry.ts list

# Output as JSON
bun ~/.claude/PAI/Tools/SessionRegistry.ts json

# Refresh then list
bun ~/.claude/PAI/Tools/SessionRegistry.ts list --refresh
```

### HTTP API

**Base URL:** `http://localhost:31337` (Pulse server)

**Endpoints:**

```bash
# Get all sessions
GET /sessions
curl http://localhost:31337/sessions

# Get specific session
GET /sessions/:id
curl http://localhost:31337/sessions/ses_abc123

# Health check
GET /health
```

**Response Format:**

```json
{
  "sessions": [
    {
      "id": "ses_abc123",
      "engine": "PNK",
      "title": "OpenCode CLI vs Web capabilities",
      "status": "active",
      "type": "interactive",
      "pane_id": "w123:p456",
      "session_url": "https://pai-primary.taild73141.ts.net/?session=ses_abc123",
      "created_at": "2026-07-05T22:00:00.000Z",
      "updated_at": "2026-07-05T22:46:00.000Z",
      "cwd": "/home/duane/pai-opencode-local"
    }
  ],
  "last_updated": "2026-07-05T22:49:39.452Z"
}
```

### Web GUI Integration

The OpenCode web interface at `https://pai-primary.taild73141.ts.net/` can fetch and display the session list via:

```javascript
// Fetch all sessions
const response = await fetch('http://localhost:31337/sessions');
const registry = await response.json();

// Display in UI
registry.sessions.forEach(session => {
  console.log(`${session.engine}: ${session.title} (${session.status})`);
  if (session.session_url) {
    console.log(`  → ${session.session_url}`);
  }
});
```

## Session Status

| Status | Meaning |
|--------|---------|
| `active` | Currently accepting input / executing |
| `background` | Session exists but not active |
| `idle` | Waiting for input, no activity |
| `blocked` | Waiting on user action or blocked |
| `done` | Execution completed |

## Session Types

| Type | Description |
|------|-------------|
| `interactive` | TUI/CLI session with user interaction |
| `exec` | Non-interactive execution (`codex exec`, `opencode run`) |
| `background` | Subagent or daemon session |
| `subagent` | Spawned by another session |

## Data Sources

The registry aggregates data from:

1. **OpenCode DB** (`~/.local/share/opencode/opencode.db`)
2. **Claude Code sessions** (`~/.claude/MEMORY/STATE/session-names.json`)
3. **Herdr panes** (via HerdrSocket API)
4. **Background agents** (future: subagent tracking)

## Files

| File | Purpose |
|------|---------|
| `~/.claude/PAI/Tools/SessionRegistry.ts` | Core registry builder |
| `~/.opencode/commands/sessions.ts` | TUI `/sessions` command |
| `~/PAI/PULSE/modules/sessions-api.ts` | HTTP API module |
| `~/.claude/MEMORY/STATE/unified-session-registry.json` | Cached registry data |

## Integration Points

### Pulse Server

The sessions API is integrated into the Pulse server (`pulse.ts`) on port 31337:

- `/sessions` → List all sessions
- `/sessions/:id` → Get session detail

### OpenCode Web

Add to the web GUI navigation:

```html
<a href="/sessions">📋 Sessions</a>
```

Fetch and display:

```javascript
const sessions = await (await fetch('/sessions')).json();
```

### Herdr

Herdr-managed panes automatically appear in the registry with their status and pane IDs.

## Future Enhancements

- [ ] Real-time session updates via WebSocket
- [ ] Session kill/resume controls via API
- [ ] Background subagent tracking
- [ ] Session activity graphs
- [ ] Cross-engine handoff visualization
- [ ] Session cost/token tracking
- [ ] Session duration statistics

## Troubleshooting

**Empty registry:**
```bash
# Force rebuild
bun ~/.claude/PAI/Tools/SessionRegistry.ts build
```

**API not responding:**
```bash
# Check Pulse server
systemctl --user status pai-pulse.service

# Restart Pulse
systemctl --user restart pai-pulse.service
```

**OpenCode sessions missing:**
```bash
# Verify DB exists
ls -lh ~/.local/share/opencode/opencode.db

# Test direct query
opencode session list
```

**Herdr sessions not showing:**
```bash
# Check Herdr is running
bun ~/.claude/PAI/Tools/HerdrCockpit.ts status
```
