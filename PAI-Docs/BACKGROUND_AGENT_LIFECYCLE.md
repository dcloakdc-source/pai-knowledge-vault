# Background Agent Lifecycle Management

**Status**: Production-ready (2026-07-06)
**Issue**: Background agents don't automatically close after task completion

## Problem

Background agents spawned with `run_in_background: true` are implemented as team members in Claude Code's implicit team system. After completing their tasks, these agents:

1. **Persist at a prompt** waiting for more work
2. **Consume memory** and show up in `ps` listings
3. **Remain in tmux panes** until manually closed
4. **Have no automatic cleanup** mechanism

This creates "zombie" agents that accumulate over time, requiring manual intervention.

## Root Cause

When Claude Code removed explicit team management (`TeamCreate`/`TeamDelete`), every session gained an implicit team. Background agents are spawned as teammates and persist indefinitely, waiting for additional `SendMessage` commands or tasks.

The `SubagentStop` hook fires when agents complete, but there was no cleanup logic to close idle team members.

## Solution

### BackgroundAgentManager Tool

**Location**: `~/.claude/PAI/Tools/BackgroundAgentManager.ts`

CLI tool for managing background agent lifecycle:

```bash
# Check current status
bun BackgroundAgentManager.ts status

# List all background agents with details
bun BackgroundAgentManager.ts list

# Clean up idle agents (graceful exit via tmux)
bun BackgroundAgentManager.ts cleanup

# Force kill all background agents
bun BackgroundAgentManager.ts cleanup --force

# Run as daemon (continuous monitoring)
bun BackgroundAgentManager.ts watch
```

### BackgroundAgentCleanup Hook

**Location**: `~/.claude/hooks/BackgroundAgentCleanup.hook.ts`
**Trigger**: `SubagentStop` event

Automatically cleanup idle background agents after any agent completes.

**Configuration**:
```bash
# Enable auto-cleanup
export PAI_BACKGROUND_AGENT_CLEANUP=1

# Set idle threshold (default: 5 minutes)
export PAI_BACKGROUND_AGENT_IDLE_MINUTES=5
```

Add to `~/.claude/settings.json`:
```json
{
  "hooks": [
    {
      "event": "SubagentStop",
      "command": "bun ~/.claude/hooks/BackgroundAgentCleanup.hook.ts"
    }
  ]
}
```

## Detection Logic

An agent is considered **idle** if:
- No entries in `agent-activity.jsonl` for its parent session in last 5 minutes
- Process still running but waiting at prompt
- No active tool calls or user interaction

An agent is considered **active** if:
- Recent activity logged in last 5 minutes
- Currently executing tools
- User is actively interacting

## Cleanup Process

1. **Graceful exit** (preferred):
   - Locate agent's tmux pane via PID mapping
   - Send `exit` command via `tmux send-keys`
   - Wait 1 second for clean shutdown

2. **Force termination** (fallback):
   - Send TERM signal if graceful exit fails
   - Used only with `--force` flag or if agent unresponsive

## Integration Points

### Manual Cleanup

Call from command line or Algorithm VERIFY phase:

```bash
bun ~/.claude/PAI/Tools/BackgroundAgentManager.ts cleanup
```

### Automatic Cleanup

Enable the SubagentStop hook in settings.json:

```json
{
  "hooks": [
    {
      "event": "SubagentStop",
      "command": "bun ~/.claude/hooks/BackgroundAgentCleanup.hook.ts"
    }
  ]
}
```

Set environment variable:
```bash
export PAI_BACKGROUND_AGENT_CLEANUP=1
```

### Daemon Mode

Run as systemd service for continuous monitoring:

```bash
# Test daemon mode
bun ~/.claude/PAI/Tools/BackgroundAgentManager.ts watch

# Create systemd service (example)
[Unit]
Description=PAI Background Agent Cleanup Daemon
After=network.target

[Service]
Type=simple
User=duane
ExecStart=/home/duane/.bun/bin/bun /home/duane/.claude/PAI/Tools/BackgroundAgentManager.ts watch
Restart=always
RestartSec=60

[Install]
WantedBy=multi-user.target
```

## Monitoring

### Check Status

```bash
# Quick status check
bun BackgroundAgentManager.ts status

# Output:
# Background Agent Status:
#   Total: 9
#   Active: 0
#   Idle: 9
#
# Idle agents can be cleaned up with: bun BackgroundAgentManager.ts cleanup
```

### List Details

```bash
bun BackgroundAgentManager.ts list

# Output:
# Found 9 background agent(s):
#
# 💤 IDLE anthropic-sources (Intern)
#   PID: 3339051 | Team: session-4488423c | Pane: %0
#   Model: sonnet | Parent: fb576884...
#
# ...
```

### Activity Log

Background agent activity is logged to:
- `~/MEMORY/STATE.claude/agent-activity.jsonl` - Agent actions
- `~/MEMORY/STATE.claude/agent-launches.jsonl` - Spawn events
- `~/.claude/hooks/subagent-stop-debug.log` - Completion events

## Known Limitations

1. **Team-only**: Only works for agents spawned as team members (the default)
2. **Tmux dependency**: Graceful exit requires tmux socket access
3. **Heuristic idle detection**: Based on activity log timing, not definitive
4. **No rollback**: Closed agents cannot be restarted

## Best Practices

1. **Enable auto-cleanup** for interactive sessions with many background agents
2. **Manual cleanup** for production workflows where agent persistence is desired
3. **Force flag** only when agents are truly stuck (not just idle)
4. **Monitor status** periodically during long-running Algorithm sessions

## Future Enhancements

Potential improvements:

1. **Configurable idle threshold** per agent type
2. **Agent checkpointing** before cleanup for resumability
3. **Integration with AgentLoopMonitor** for better activity tracking
4. **Cleanup policies** (keep N most recent, close after X hours, etc.)
5. **Agent parking** (suspend instead of terminate)

## References

- Delegation skill: `~/.claude/skills/Delegation/SKILL.md`
- AgentStart hook: `~/.claude/hooks/AgentStart.hook.ts`
- AgentOutputCapture: `~/.claude/hooks/AgentOutputCapture.hook.ts`
- Agent loop observability: `~/.claude/PAI/DOCUMENTATION/AGENT_LOOPS.md`
