# PAI Engine Daemon Architecture

**Version:** 1.0  
**Updated:** 2026-06-24  
**Author:** PAI Nova (PNK)

## Overview

PAI's multi-engine architecture currently runs **infrastructure services** as systemd daemons while AI engines (PNC/Claude Code, PNG/Antigravity, PNX/Codex, PNK/OpenCode) operate as **interactive terminal sessions**. This document explores the architectural tradeoffs, current state, and potential daemon patterns for AI engines.

---

## Current State

### What Runs as Daemons (47 systemd user units)

#### Tier 1 Services (RTO: 60s)
- **OmniPulse Gateway** (`pai-pulse.service`) — Port 31337, unified daemon orchestrator
- **Functions API** (`pai-functions.service`) — Port 8890, JIT credential gateway
- **Command Center** (`pai-command-center.service`) — Port 8766, web dashboard
- **Ollama** — Port 11434, local inference (system-level service)

#### Tier 2 Services (RTO: 5m)
- **Memory SSE** (`pai-memory-sse.service`) — Port 8891, MCP bridge
- **Voice Server** (`pai-voiceserver.service`) — Port 8888, TTS notifications
- **Skill Dispatcher** (`pai-skill-dispatcher.service`) — Background skill execution
- **Switchboard Bridge** (`pai-switchboard-bridge.service`) — Inter-agent messaging
- **Local Council** (`pai-local-council.service`) — Multi-model debate server
- **DA Bridge** (`pai-da-bridge.service`) — Synchronous DA reply endpoint
- **Brain Ingest** (`pai-brain-ingest.service`) — Antigravity → PAI memory sync
- **Litestream** (`pai-litestream.service`) — Database replication
- **Log Scrubber** (`pai-log-scrubber.service`) — SIEM log processing
- **Inbox Watchdog** (`pai-inbox-watchdog.service`) — Cross-engine task notifications
- **Egress Monitor** (`pai-egress-monitor.service`) — Network egress auditing
- **Canary Listener** (`pai-canary-listener.service`) — Security auditd consumer
- **Model Viewer** (`pai-model-viewer.service`) — 3D model server for Spatial3D

#### Tier 3 Services (RTO: 30m)
- **Automation jobs** (`pai-automation@*.service`) — 11 templated automation units
- **Scheduled tasks** (timers) — Deadman, backups, model discovery, benchmarks

### What Runs as Interactive Sessions

#### AI Engines (Terminal Sessions)
```bash
# PNC - Claude Code (primary DA on pai-primary)
claude                      # pts/25, 12.35min CPU over 3 days
claude                      # pts/12, 31.20min CPU over 3 days

# PNX - OpenAI Codex (autonomous execution, sandbox)
codex resume <session-id>   # pts/17, 77.42min CPU over 3 days
codex resume <session-id>   # pts/18, 70.06min CPU over 3 days

# PNK - OpenCode (provider-agnostic, multi-file builder)
opencode                    # pts/51, 26.28min CPU over 2 days

# PNG - Antigravity (Gemini research)
agy --dangerously-skip-permissions -p "..."  # ephemeral, called via Bash
```

#### Why Engines Are NOT Daemons

1. **Interactive nature** — Engines require terminal I/O, user approval, real-time editing
2. **Session context** — Each conversation is stateful, context-dependent
3. **Human-in-loop** — Engines are designed for collaborative work, not autonomous background tasks
4. **Tool access** — Engines manipulate files, git, processes; daemonizing breaks assumptions
5. **Error handling** — Terminal sessions allow human intervention on errors
6. **OAuth/auth** — Claude Code uses OAuth flow that requires browser interaction

---

## Daemon Patterns for Engines

While full daemonization doesn't fit the interactive model, there are useful **hybrid patterns**:

### Pattern 1: Long-Running Background Agent (Implemented)

**Use Case:** Autonomous tasks that don't need human approval  
**Example:** `pai-inbox-auto-processor.service` (failed, needs fixing)

```ini
[Unit]
Description=PAI Inbox Auto-Processor (Research Skill Automation)
After=network.target

[Service]
Type=simple
WorkingDirectory=/home/duane/.claude
ExecStart=/home/duane/.bun/bin/bun PAI/Tools/InboxAutoProcessor.ts
Restart=on-failure
RestartSec=300
Environment=PATH=/home/duane/.bun/bin:/usr/local/bin:/usr/bin:/bin
Environment=PAI_DIR=/home/duane/.claude
EnvironmentFile=/home/duane/.config/PAI/secrets.env

[Install]
WantedBy=default.target
```

**Limitations:**
- Cannot use interactive tools (Browser, Question, manual git)
- No user approval flow
- Limited to programmatic tasks

### Pattern 2: Tmux/Screen Persistent Sessions (Used for Codex)

**Use Case:** Resume-able terminal sessions that survive disconnection

```bash
# Start in tmux
tmux new -s pnx-worker-1 "codex resume 019ede5a-811b-7740-8d75-20387d750c3b"

# Detach and reattach later
tmux attach -t pnx-worker-1
```

**Advantages:**
- Full terminal interactivity preserved
- Can detach/reattach without losing state
- Standard systemd monitoring via PID

**Could be daemonized as:**
```ini
[Service]
Type=forking
ExecStart=/usr/bin/tmux new-session -d -s pnx-worker "codex resume 019ede5a..."
ExecStop=/usr/bin/tmux kill-session -t pnx-worker
```

### Pattern 3: HTTP/WebSocket API Server (Best for stateless work)

**Use Case:** Convert engine capabilities into callable services  
**Example:** PAI Functions API, Local Council Server

This is how you **actually daemonize** engine capabilities:

```typescript
// server.ts - wrap engine in HTTP API
import { Anthropic } from "@anthropic-ai/sdk";

const anthropic = new Anthropic();

Bun.serve({
  port: 8890,
  async fetch(req) {
    const { prompt, model = "claude-sonnet-4-6" } = await req.json();
    
    const message = await anthropic.messages.create({
      model,
      max_tokens: 8000,
      messages: [{ role: "user", content: prompt }],
    });
    
    return Response.json({ 
      response: message.content[0].text 
    });
  },
});
```

**Systemd unit:**
```ini
[Service]
Type=simple
WorkingDirectory=/home/duane/.claude/PAI/Tools/EngineAPI
ExecStart=/home/duane/.bun/bin/bun server.ts
Restart=always
EnvironmentFile=/home/duane/.config/PAI/secrets.env
```

**This pattern powers:**
- Functions API (port 8890) — stateless LLM calls
- Local Council (port varies) — multi-model debate
- DA Bridge — synchronous DA replies for Matrix bot

### Pattern 4: Cron/Timer-Triggered One-Shots (Implemented extensively)

**Use Case:** Scheduled background tasks  
**Example:** 15 active systemd timers

```ini
# pai-engine-bench-smoke.timer
[Unit]
Description=Run PAI EngineBench smoke benchmark daily

[Timer]
OnCalendar=daily
Persistent=true
RandomizedDelaySec=1h

[Install]
WantedBy=timers.target
```

```ini
# pai-engine-bench-smoke.service
[Unit]
Description=PAI EngineBench daily smoke benchmark

[Service]
Type=oneshot
WorkingDirectory=/home/duane/.claude
ExecStart=/home/duane/.bun/bin/bun PAI/Tools/EngineBench/run.ts --mode smoke
Environment=PAI_DIR=/home/duane/.claude
EnvironmentFile=/home/duane/.config/PAI/secrets.env
```

---

## Architecture Decision: Interactive vs Daemon

| Concern | Interactive (Current) | Daemon | Hybrid |
|---------|----------------------|--------|--------|
| **Human approval** | ✅ Native | ❌ Requires queue/notification | ⚠️ API + approval endpoint |
| **Terminal tools** | ✅ Full access | ❌ Headless only | ⚠️ Tmux pseudo-TTY |
| **Session context** | ✅ Conversation state | ❌ Stateless per request | ⚠️ DB-backed sessions |
| **OAuth/browser** | ✅ Native flow | ❌ Manual token refresh | ⚠️ Pre-auth + rotation |
| **File editing** | ✅ Real-time | ⚠️ Batch/delayed | ✅ Via API |
| **Error recovery** | ✅ Human intervention | ⚠️ Auto-retry only | ⚠️ Dead-letter queue |
| **Monitoring** | ❌ Manual ps/tmux | ✅ systemd integration | ✅ Best of both |
| **Auto-restart** | ❌ Manual | ✅ systemd policy | ✅ Tmux + systemd |
| **Resource limits** | ❌ Shell ulimit | ✅ systemd cgroups | ✅ Best of both |
| **Logging** | ⚠️ Terminal scrollback | ✅ journald | ✅ Both |

**Verdict:** Interactive sessions are correct for the primary DA workflow. Daemons are correct for:
1. Infrastructure (Pulse, Functions, Memory)
2. Stateless API wrappers (Council, DA Bridge)
3. Scheduled/background tasks (automation, backups, benchmarks)
4. Inter-agent messaging (Switchboard, Inbox Watchdog)

---

## Daemonization Recipes

### Recipe 1: Background Autonomous Agent

**When to use:** Long-running task with no human interaction  
**Examples:** Log scrubber, inbox watcher, canary listener

```bash
# 1. Create systemd user unit
cat > ~/.config/systemd/user/pai-my-agent.service <<'EOF'
[Unit]
Description=PAI My Agent — autonomous background task
After=network.target

[Service]
Type=simple
WorkingDirectory=/home/duane/.claude
ExecStart=/home/duane/.bun/bin/bun PAI/Tools/MyAgent.ts
Restart=on-failure
RestartSec=30
Environment=PATH=/home/duane/.bun/bin:/usr/local/bin:/usr/bin:/bin
Environment=PAI_DIR=/home/duane/.claude
EnvironmentFile=/home/duane/.config/PAI/secrets.env
StandardOutput=journal
StandardError=journal

# Resource limits
MemoryMax=1G
CPUQuota=50%

[Install]
WantedBy=default.target
EOF

# 2. Reload systemd and enable
systemctl --user daemon-reload
systemctl --user enable --now pai-my-agent.service

# 3. Monitor
systemctl --user status pai-my-agent.service
journalctl --user -u pai-my-agent.service -f
```

### Recipe 2: Tmux-Wrapped Interactive Session

**When to use:** Need terminal interactivity but want systemd management  
**Examples:** Long-running Codex sessions

```bash
# 1. Create wrapper script
cat > ~/.claude/PAI/bin/codex-worker-1 <<'EOF'
#!/usr/bin/env bash
exec tmux new-session -s codex-worker-1 \
  "codex resume 019ede5a-811b-7740-8d75-20387d750c3b"
EOF
chmod +x ~/.claude/PAI/bin/codex-worker-1

# 2. Create systemd unit
cat > ~/.config/systemd/user/pai-codex-worker-1.service <<'EOF'
[Unit]
Description=PAI Codex Worker 1 (Tmux Session)
After=network.target

[Service]
Type=forking
ExecStart=/usr/bin/tmux new-session -d -s codex-worker-1 \
  "/home/duane/.claude/PAI/bin/codex-worker-1"
ExecStop=/usr/bin/tmux kill-session -t codex-worker-1
Restart=on-failure
RestartSec=60
Environment=PATH=/home/duane/.npm-global/bin:/usr/local/bin:/usr/bin:/bin
EnvironmentFile=/home/duane/.config/PAI/secrets.env

[Install]
WantedBy=default.target
EOF

systemctl --user daemon-reload
systemctl --user enable --now pai-codex-worker-1.service

# 3. Attach to session
tmux attach -t codex-worker-1
```

### Recipe 3: HTTP API Wrapper

**When to use:** Stateless engine calls from other services  
**Examples:** Functions API, Council Server

```typescript
// PAI/Tools/EngineAPI/server.ts
import { Anthropic } from "@anthropic-ai/sdk";

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const server = Bun.serve({
  port: parseInt(process.env.PAI_ENGINE_PORT || "8892"),
  
  async fetch(req: Request) {
    if (req.method !== "POST") {
      return new Response("Method not allowed", { status: 405 });
    }

    try {
      const { prompt, model = "claude-sonnet-4-6", max_tokens = 8000 } = await req.json();

      const message = await anthropic.messages.create({
        model,
        max_tokens,
        messages: [{ role: "user", content: prompt }],
      });

      return Response.json({
        success: true,
        response: message.content[0].type === "text" 
          ? message.content[0].text 
          : null,
        usage: message.usage,
      });
    } catch (error) {
      console.error("Engine API error:", error);
      return Response.json(
        { success: false, error: String(error) },
        { status: 500 }
      );
    }
  },
});

console.log(`PAI Engine API listening on :${server.port}`);
```

```ini
# ~/.config/systemd/user/pai-engine-api.service
[Unit]
Description=PAI Engine API — HTTP wrapper for Claude
After=network.target

[Service]
Type=simple
WorkingDirectory=/home/duane/.claude/PAI/Tools/EngineAPI
ExecStart=/home/duane/.bun/bin/bun server.ts
Restart=always
RestartSec=10
Environment=PATH=/home/duane/.bun/bin:/usr/local/bin:/usr/bin:/bin
Environment=PAI_DIR=/home/duane/.claude
Environment=PAI_ENGINE_PORT=8892
EnvironmentFile=/home/duane/.config/PAI/secrets.env
StandardOutput=journal
StandardError=journal

# Health check
ExecStartPost=/bin/sleep 2
ExecStartPost=/usr/bin/curl -f http://localhost:8892/health || exit 1

[Install]
WantedBy=default.target
```

### Recipe 4: Scheduled One-Shot (Timer)

**When to use:** Periodic background task  
**Examples:** Backups, benchmarks, model discovery

```bash
# Service unit (one-shot)
cat > ~/.config/systemd/user/pai-my-task.service <<'EOF'
[Unit]
Description=PAI My Task — scheduled background job

[Service]
Type=oneshot
WorkingDirectory=/home/duane/.claude
ExecStart=/home/duane/.bun/bin/bun PAI/Tools/MyTask.ts
Environment=PAI_DIR=/home/duane/.claude
EnvironmentFile=/home/duane/.config/PAI/secrets.env
StandardOutput=journal
StandardError=journal
EOF

# Timer unit
cat > ~/.config/systemd/user/pai-my-task.timer <<'EOF'
[Unit]
Description=Run PAI My Task daily at 3am

[Timer]
OnCalendar=daily
OnCalendar=*-*-* 03:00:00
Persistent=true
RandomizedDelaySec=30m

[Install]
WantedBy=timers.target
EOF

systemctl --user daemon-reload
systemctl --user enable --now pai-my-task.timer

# Check timer status
systemctl --user list-timers pai-my-task.timer
```

---

## Monitoring and Observability

### Health Checks

All daemon services should expose health endpoints where appropriate:

```typescript
// Add to any Bun server
if (req.url.endsWith("/health")) {
  return Response.json({
    status: "healthy",
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    timestamp: new Date().toISOString(),
  });
}
```

### Systemd Integration

```bash
# Service status
systemctl --user status pai-*.service

# Failed services
systemctl --user --failed

# Resource usage
systemd-cgtop --user

# Journal logs (last 100 lines)
journalctl --user -u pai-pulse.service -n 100

# Follow logs live
journalctl --user -u pai-pulse.service -f

# Logs since boot
journalctl --user -u pai-pulse.service -b

# Logs for time range
journalctl --user -u pai-pulse.service --since "2026-06-24 00:00" --until "2026-06-24 12:00"
```

### Observability Dashboard

The PAI Observability Server (`pai-observability.service`) provides a unified view:

```bash
# Web dashboard
curl http://localhost:8766/

# CLI probe
bun ~/.claude/PAI/Tools/HA/ha-status.ts
```

### Alerting

Dead services can be detected by:

1. **Deadman timer** (`pai-deadman.timer`) — checks every 30min, probes all tracked services
2. **Health probes** — HTTP checks via `pai-container-healthprobe.service`
3. **Voice notifications** — failures trigger TTS alerts via `pai-voiceserver.service`

---

## Anti-Patterns to Avoid

### ❌ Don't daemonize the primary DA

The primary Claude Code session (PNC on pai-primary) should remain interactive. It's your main interface to PAI, handles Algorithm runs, and requires approval flows.

### ❌ Don't run engines without restart policies

If you must daemonize an engine, always set `Restart=on-failure` or `Restart=always` with `RestartSec`.

### ❌ Don't share credentials across daemon and interactive

Daemons load from `EnvironmentFile=/home/duane/.config/PAI/secrets.env`.  
Interactive sessions may use OAuth or different credential sources.  
Never mix the two.

### ❌ Don't run daemons as root

All PAI services use **user systemd** (`systemctl --user`), never system-wide units, except for:
- `docvault-sync.service` (system, runs as root for NAS mount)
- `pai-nas-route.service` (system, network routing)

### ❌ Don't skip resource limits

Always set at minimum:
```ini
[Service]
MemoryMax=1G        # Prevent runaway memory
CPUQuota=50%        # Limit CPU usage
RestartSec=30       # Backoff on failure
```

### ❌ Don't ignore failed dependencies

Use `After=` and `Wants=` to declare dependencies:

```ini
[Unit]
After=network-online.target pai-memory.service
Wants=network-online.target
```

---

## Integration with PAI Architecture

### How Daemons Fit Into PAI

```
┌─────────────────────────────────────────────────────────────┐
│                     PAI Architecture                         │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  INTERACTIVE LAYER (Human-in-loop)                           │
│  ┌──────────────────────────────────────────────────┐       │
│  │ PNC (Claude Code) - Primary DA                   │       │
│  │ PNG (Antigravity) - Research                     │       │
│  │ PNX (Codex) - Autonomous execution (tmux)        │       │
│  │ PNK (OpenCode) - Multi-file builder              │       │
│  └──────────────────────────────────────────────────┘       │
│                        ↕                                     │
│  DAEMON LAYER (Background services)                          │
│  ┌──────────────────────────────────────────────────┐       │
│  │ Tier 1 (RTO 60s)                                 │       │
│  │  • OmniPulse (orchestration)                     │       │
│  │  • Functions API (stateless LLM)                 │       │
│  │  • Command Center (dashboard)                    │       │
│  │  • Ollama (local inference)                      │       │
│  │                                                   │       │
│  │ Tier 2 (RTO 5m)                                  │       │
│  │  • Memory SSE (MCP bridge)                       │       │
│  │  • Voice Server (TTS)                            │       │
│  │  • Skill Dispatcher                              │       │
│  │  • Switchboard Bridge (inter-agent messaging)    │       │
│  │  • Local Council (multi-model debate)            │       │
│  │  • DA Bridge (Matrix bot endpoint)               │       │
│  │                                                   │       │
│  │ Tier 3 (RTO 30m)                                 │       │
│  │  • Litestream (DB replication)                   │       │
│  │  • Log Scrubber (SIEM)                           │       │
│  │  • Automation jobs (11 templated units)          │       │
│  │  • Scheduled tasks (15 timers)                   │       │
│  └──────────────────────────────────────────────────┘       │
│                        ↕                                     │
│  DATA LAYER (Persistent state)                               │
│  ┌──────────────────────────────────────────────────┐       │
│  │ • SQLite (Litestream-replicated)                 │       │
│  │ • JSONL logs (MEMORY/STATE.claude/)              │       │
│  │ • NAS backups (qnap-nas:/share/PAI/)             │       │
│  │ • Entity graph (Qdrant)                          │       │
│  └──────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────┘
```

### Cross-Engine Communication via Daemons

Engines don't talk directly to each other. They use daemon services as message buses:

1. **Switchboard Bridge** (`pai-switchboard-bridge.service`)  
   - Watches `MEMORY/STATE.claude/agent-inbox/{pnc,pnx,png,pnk}/`
   - Routes messages between engines
   - Voice notifications on new tasks

2. **Functions API** (`pai-functions.service`)  
   - HTTP gateway for token-free LLM calls
   - Used by Antigravity (PNG) and Telegram bot
   - Forwards voice requests to Voice Server

3. **DA Bridge** (`pai-da-bridge.service`)  
   - Synchronous real-DA reply endpoint
   - Used by Matrix bot for instant responses
   - Queues complex tasks for PNC pickup

4. **OmniPulse** (`pai-pulse.service`)  
   - Unified daemon orchestrator
   - Cockpit dashboard at :31337
   - Hooks, health checks, observability

---

## Future Considerations

### Warm Standby for Critical Services

Per `PAI/INFRA/HA_REDUNDANCY.md`, next steps include:

1. **M710q warm standby** for Functions API, Command Center
2. **Legion secondary** for Ollama inference failover
3. **Proxmox VM** for full control-plane redundancy

This would involve:
- Service discovery (Consul/etcd or Tailscale DNS)
- Automatic failover (keepalived or custom health probe)
- Shared state (Litestream replicas, distributed SQLite)

### Agent Pool Pattern

Instead of fixed Codex sessions, consider a **worker pool**:

```
┌─────────────────────────────────────┐
│ Agent Pool Manager (daemon)         │
├─────────────────────────────────────┤
│ • Spawns N Codex workers in tmux    │
│ • Assigns tasks from queue          │
│ • Monitors health, restarts failed  │
│ • Scales up/down based on load      │
└─────────────────────────────────────┘
         ↓
   ┌────────────────┐
   │ Task Queue     │
   │ (Redis/SQLite) │
   └────────────────┘
```

### Kubernetes/Docker?

PAI explicitly avoids containerization complexity on the primary host. Systemd user units provide:
- Resource limits (cgroups)
- Auto-restart
- Dependency management
- Logging (journald)
- Health checks

Without the overhead of:
- Container runtime
- Image registry
- Network complexity
- Volume mounts

For multi-host deployments, **Tailscale + systemd** is the current pattern. Kubernetes may make sense for:
- Multi-tenant PAI (NOT the design — see `PAI_SYSTEM_PROMPT.md` single-beneficiary rule)
- Cloud deployment (but PAI is local-first by design)

---

## Summary

### Current Best Practices

✅ **Do daemonize:**
- Infrastructure (Pulse, Functions, Memory)
- Stateless API wrappers (Council, DA Bridge)
- Background tasks (log scrubbing, inbox watching)
- Scheduled jobs (backups, benchmarks)

✅ **Keep interactive:**
- Primary DA (PNC on pai-primary)
- Research sessions (PNG/Antigravity)
- Multi-file builders (PNK/OpenCode)
- Autonomous agents that need approval (Algorithm runs)

✅ **Hybrid approach for:**
- Long-running Codex workers (tmux + systemd)
- Background agents with occasional human input (queue + notification)

### Service Count

As of 2026-06-24:
- **47 systemd user units** installed
- **25 active services** running now
- **15 active timers** for scheduled tasks
- **0 daemonized AI engines** (all interactive)

### Key Insight

PAI's strength is **human-AI collaboration**. Daemonizing everything would break that. The current hybrid model — daemons for infrastructure, interactive for intelligence — is architecturally sound.

---

## Appendix: Complete Service Inventory

### Active Services (Running Now)

```
pai-brain-ingest.service           - Antigravity brain → memory sync
pai-canary-listener.service        - Security auditd consumer
pai-command-center.service         - Web dashboard :8766
pai-container-healthprobe.service  - External container health probe
pai-da-bridge.service              - Matrix bot DA endpoint
pai-egress-monitor.service         - Network egress audit
pai-engine-bench-smoke.service     - Daily benchmark (oneshot)
pai-engine-bench-weekly.service    - Weekly multi-engine benchmark (oneshot)
pai-engine-meter.service           - Token/cost rollup (oneshot)
pai-entity-graph.service           - Nightly graph rebuild (oneshot)
pai-fleet-reaper.service           - Tmux session cleanup (oneshot)
pai-functions.service              - Functions API :8890
pai-inbox-watchdog.service         - Cross-engine task notifications
pai-litestream.service             - SQLite replication
pai-local-council.service          - Multi-model debate server
pai-log-scrubber.service           - SIEM log processing
pai-memory-sse.service             - MCP SSE bridge :8891
pai-model-viewer.service           - 3D model server (Spatial3D)
pai-observability.service          - Life Dashboard
pai-pulse.service                  - OmniPulse :31337
pai-pxe-http.service               - PXE boot server :8077
pai-skill-dispatcher.service       - Skill execution daemon
pai-switchboard-bridge.service     - Inter-agent messaging
pai-voiceserver.service            - TTS server :8888
```

### Active Timers (Scheduled)

```
pai-automation@google-key-ip-sync.timer  - Every 15min
pai-container-healthprobe.timer          - Every 5min
pai-deadman.timer                        - Every 30min
pai-engine-bench-smoke.timer             - Daily
pai-engine-bench-weekly.timer            - Weekly
pai-engine-meter.timer                   - Daily
pai-entity-graph.timer                   - Daily
pai-fleet-reaper.timer                   - Every 30min
pai-matrix-escrow.timer                  - Weekly
pai-model-benchmark.timer                - Weekly (Sundays 4am)
pai-model-discover.timer                 - Daily (3am)
pai-perm-soak.timer                      - Weekly
pai-qdrant-backup.timer                  - Daily (3:15am)
pai-reflection-mining.timer              - Weekly
```

### Failed Services (Need Attention)

```
pai-automation@credential-usage-siem.service  - SIEM credential tracking
pai-deadman.service                           - Universal deadman check
pai-inbox-auto-processor.service              - Research automation
```

---

**Document Status:** Live reference, update as architecture evolves  
**Related Docs:**
- `PAI/INFRA/HA_REDUNDANCY.md` — HA strategy
- `PAI/Tools/HA/topology.json` — Service topology
- `PAI/DOCUMENTATION/ARCHITECTURE_SUMMARY.md` — System overview
- `PAI/PAISYSTEMARCHITECTURE.md` — Full architecture
