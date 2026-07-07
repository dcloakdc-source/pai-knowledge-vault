# PAI Session Tracking - Complete System

**Version:** 1.0.0  
**Status:** Production Ready

## The Problem We Solved

**Before:** When you asked "what sessions are active?", the system only checked one engine's registry and missed 70+ active sessions across all engines and services.

**Now:** Comprehensive tracking across ALL engines and services - Claude Code, OpenCode, Antigravity, MCP servers, PAI services, everything.

---

## Quick Usage

### Check All Active Sessions

```bash
pai --status
```

**Shows:**
- All Claude Code (PNC) sessions
- All OpenCode (PNK) sessions  
- All Council runners
- All MCP servers (49+)
- All PAI background services (13)
- Antigravity IDE server
- Development servers

**Example output:**
```
Summary:
  PNC: 3 active
  PNK: 5 active
  Council: 4 active
  MCP: 49 active
  PAI: 13 active
  PNG: 1 active
  TOTAL: 75 sessions
```

---

### Direct Session Tracker

```bash
# Basic view
bun ~/.claude/PAI/Tools/SessionTracker.ts

# Verbose (show full commands)
bun ~/.claude/PAI/Tools/SessionTracker.ts --verbose

# JSON output (for scripting)
bun ~/.claude/PAI/Tools/SessionTracker.ts --json
```

---

## What Gets Tracked

### 1. Claude Code (PNC) Sessions

**Detection:** `ps aux | grep 'claude$'`

**Shows:**
- PID
- Terminal (pts/8, pts/11, etc.)
- Runtime (24h, 53h, etc.)

**Example:**
```
PNC (3 active):
  PID 1173052 [session] - Claude Code session (24:08) @pts/8
  PID 3635879 [session] - Claude Code session (53:05) @pts/11
  PID 3964833 [session] - Claude Code session (8:57) @pts/12
```

---

### 2. OpenCode (PNK) Sessions

**Detection:** `pgrep -a opencode | grep 'run|serve'`

**Shows:**
- Server (opencode serve)
- Active sessions (opencode run)
- Council debates
- Port numbers

**Example:**
```
PNK (5 active):
  PID 12199 [server] - OpenCode server :8123
  PID 393149 [session] - OpenCode session
  PID 616146 [council] - Council debate session
```

---

### 3. UnifiedCouncil Runners

**Detection:** `pgrep -a bun | grep council-runner`

**Shows:**
- Active council debate runners
- Round processing

**Example:**
```
Council (4 active):
  PID 361633 [runner] - UnifiedCouncil runner
  PID 587791 [runner] - UnifiedCouncil runner (Round 2)
  PID 806164 [runner] - UnifiedCouncil runner (Round 3)
  PID 878469 [runner] - UnifiedCouncil runner (Round 16)
```

---

### 4. MCP Servers

**Detection:** `ps aux | grep 'pai-mcp-server|CouncilMCP|VoiceMCP|MemoryMCP|okf-mcp'`

**Shows:**
- PAI MCP servers (16+)
- Council MCP servers (5+)
- Voice MCP servers (5+)
- Memory MCP servers (4+)
- OKF MCP servers
- System MCP servers (time, git, fetch, playwright)

**Example:**
```
MCP (49 active):
  PID 22335 [mcp] - PAI MCP server (0:06)
  PID 22338 [mcp] - Council MCP server (0:09)
  PID 22346 [mcp] - Voice MCP server (0:09)
  PID 33458 [mcp] - Memory MCP server (3:36)
  PID 3965508 [mcp] - OKF MCP server (0:11)
```

---

### 5. PAI Background Services

**Detection:** Checks for specific service processes

**Services tracked:**
1. Forum dispatcher
2. Canary listener
3. Command Center (often 300+ hours runtime!)
4. DA bridge
5. Egress monitor
6. Functions API
7. Inbox watchdog
8. Log scrubber (often 1700+ hours!)
9. Skill dispatcher
10. Voice server
11. Brain ingest
12. 3D model server
13. OmniPulse gateway

**Example:**
```
PAI (13 active):
  PID 12193 [service] - Forum dispatcher (22:12)
  PID 12202 [service] - Command Center (326:29)
  PID 12212 [service] - Log scrubber (1791:29)
  PID 12207 [service] - Functions API (10:40)
```

---

### 6. Antigravity IDE Server (PNG)

**Detection:** `pgrep -a node | grep antigravity-ide-server`

**Shows:**
- Main PNG server process
- Connection count

**Example:**
```
PNG (1 active):
  PID 497356 [server] - Antigravity IDE server
```

---

### 7. Development Servers

**Detection:** `ps aux | grep 'npm run dev|next dev'`

**Shows:**
- Next.js dev servers
- Other npm dev servers
- Port numbers

**Example:**
```
Dev (1 active):
  PID 4188246 [server] - Development server :3300
```

---

## Common Scenarios

### Morning Check

```bash
pai --status
```

See everything that's running - active sessions, long-running services, background tasks.

---

### Find Long-Running Sessions

```bash
bun SessionTracker.ts | grep -E "\([0-9]+h"
```

Shows sessions running for hours/days.

---

### Count by Engine

```bash
pai --status | grep "active" | head -10
```

Quick count per engine.

---

### Kill Specific Session

```bash
# Find PID
pai --status | grep "Claude Code session"

# Kill it
kill <PID>
```

---

### JSON for Scripting

```bash
bun SessionTracker.ts --json | jq '.[] | select(.engine=="PNC")'
```

Filter to specific engine.

---

## Integration with Orchestration

### 1. `pai --status` Shows Sessions First

When you run `pai --status`, you now see:
1. **Active sessions** (comprehensive, all engines)
2. Budget status
3. Task queue

Sessions are the most important context - knowing what's running matters more than budget percentages.

---

### 2. Session Count in Dashboard

The `EngineDashboard.ts` should show active session counts per engine. (Future enhancement)

---

### 3. Budget Impact Awareness

Seeing 3 Claude Code sessions running for 50+ hours each? That's budget information you need to make routing decisions.

---

## Troubleshooting

### No sessions showing for an engine

**Check manually:**
```bash
# Claude
ps aux | grep claude

# OpenCode
pgrep -a opencode

# Antigravity
ps aux | grep antigravity
```

If processes exist but don't show up, the detection regex may need updating.

---

### "75 sessions" seems high

**That's normal!** Most are:
- MCP servers (3-4 per Claude session)
- Background services (always running)
- Long-lived sessions (pts/11 at 50+ hours is one session, not new)

**Actual interactive sessions:** Usually 3-6 (PNC + PNK)

---

### Want to see full commands

```bash
bun SessionTracker.ts --verbose
```

Shows the complete command line for each process.

---

## Files

```
PAI/Tools/
├── SessionTracker.ts          # Comprehensive session tracker
└── pai                        # Unified CLI (calls SessionTracker)

PAI/DOCUMENTATION/
└── SESSION_TRACKING.md        # This file
```

---

## Future Enhancements

### Planned

1. **Session grouping** - Group MCP servers by parent session
2. **Activity detection** - Show CPU/memory for each session
3. **Session history** - Track session start/end times
4. **Auto-cleanup** - Suggest zombie sessions to kill
5. **Dashboard integration** - Live session count in EngineDashboard

### Considered

6. **Session context** - Show what each session is working on
7. **Cost attribution** - Estimate cost per active session
8. **Session tagging** - Label sessions by project/task
9. **Kill groups** - Kill all sessions for a specific engine
10. **Session restore** - Resume sessions after system restart

---

## Summary

**Before:** "What sessions are active?" → Missed 70+ sessions

**Now:** Complete visibility across all engines and services

**Command:** `pai --status` or `bun SessionTracker.ts`

**Output:** Grouped by engine, showing PID, type, description, runtime, terminal, port

**Integrated:** Part of the unified PAI orchestration system

---

*Last updated: 2026-07-05*
