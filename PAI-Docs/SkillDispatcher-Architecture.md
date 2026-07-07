# Skill Dispatcher Daemon — Architecture (v1.0)

**Status:** Design complete, build pending  
**Backlog refs:** #11 (run_skill), #12 (spawn_agent), #13 (daemon)  
**Date:** 2026-05-22

---

## 1. Problem Statement

PAI has 94 skills and 32 agents, but they can only be invoked through Claude Code. Other engines — Ollama, Gemini CLI, Codex CLI, Discord bot, Telegram bot, iMessage — have no way to leverage PAI's skill surface.

Currently, the Discord adapter has an inline route_to_engine method that spawns subprocesses. This is fragile (each caller duplicates subprocess management), unbalanced (only ollama/gemini/codex), unscalable (6 adapters lack support), and skill-unaware (raw prompts, no skill mapping).

A centralized daemon solves all of these with one HTTP API.

---

## 2. Architecture Overview

```
                        +------------------------------------------+
                        |        SkillDispatcher Daemon            |
                        |         (Bun.serve :31340)               |
                        |                                          |
 Discord bot ---------->|  POST /run_skill  --> EngineRouter -->   |
 Gemini CLI ----------->|  POST /spawn_agent -> EngineRouter       |
 Codex CLI ------------>|  GET  /health                            |
 Ollama A2A ----------->|  GET  /engines                           |
 Telegram bot --------->|                                          |
 Slack bot ------------>|  JWT Auth <-- OmniPulse PKI             |
 iMessage ------------->|  Rate Limiter <-- per-caller tracking   |
                        +------------------------------------------+
                                                    |
                         +--------------------------+
                         v
             +-----------------------+
             |    Engine Invocation   |
             +-----------------------+
             | claude subprocess      |
             | opencode subprocess    |
             | gemini subprocess      |
             | codex subprocess       |
             | ollama HTTP API        |
             +-----------------------+
```

**Binding:** 127.0.0.1:31340 — local-only, no network exposure.  
**Auth:** JWT via OmniPulse PKI.  
**Transport:** HTTP/1.1 JSON with Bun.serve.

---

## 3. API Endpoints

### 3.1 POST /run_skill

Dispatch a PAI skill to an engine.

**Request fields:**
- skill (required): Skill name or inference-matrix task profile
- prompt (required): The task text
- context (optional): Extra context object
- engine_preference (optional): "claude", "opencode", "gemini", "codex", "ollama", "auto" (default)
- timeout_ms (optional): Max wait, default 60000, cap 300000
- caller (optional): Identifier for rate limiting and logging

**Success response (200):**
- status: "ok"
- skill: echoed skill name
- engine_used: which engine handled it
- result: the engine's output text
- elapsed_ms: wall-clock duration
- tokens: { input, output } (where available)
- caller: echoed

**Error response:**
- status: "error"
- error: message explaining what failed
- skill: echoed
- elapsed_ms: time before failure

### 3.2 POST /spawn_agent

Spawn a PAI agent through Claude Code or OpenCode.

**Request fields:**
- agent_type (required): Agent name matching agent registry
- prompt (required): The agent task
- model (optional): "sonnet", "haiku", "opus", "auto"
- context (optional): { working_dir, skills[] }
- timeout_ms (optional): Default 300000 (5 min)
- caller (optional): Identifier

**Response (200):**
- status: "ok"
- agent_type: echoed
- engine_used: "claude" or "opencode"
- result: agent output
- session_id: subagent session ID
- elapsed_ms: duration

### 3.3 GET /health

Returns uptime, available engines, request/error counts, version.

### 3.4 GET /engines

Returns per-engine status with paths and versions.

---

## 4. Skill Registry

JSON file at PAI/Tools/SkillDispatcher/skill-registry.json:

```json
{
  "Research": {
    "engines": ["claude", "gemini", "codex"],
    "default_engine": "claude",
    "timeout_ms": 60000,
    "type": "skill"
  },
  "OllamaSkill": {
    "engines": ["ollama"],
    "default_engine": "ollama",
    "timeout_ms": 30000,
    "type": "inference"
  },
  "multi_step_reasoning": {
    "engines": ["ollama", "claude"],
    "default_engine": "ollama",
    "timeout_ms": 120000,
    "type": "inference"
  },
  "fast_classification": {
    "engines": ["ollama"],
    "default_engine": "ollama",
    "timeout_ms": 10000,
    "type": "inference"
  }
}
```

Unknown skills default to engines: ["claude"], 60s timeout. Daemon logs warnings for registry gap analysis.

---

## 5. Engine Invocation

| Engine | Command pattern | Output cleaning | Path probe |
|--------|----------------|-----------------|------------|
| claude | claude -p "..." --output-format text --max-turns 1 | Raw (trimmed) | ~/.local/bin/claude |
| opencode | opencode run "..." | Raw | ~/.local/bin/opencode |
| gemini (PNG) | agy --dangerously-skip-permissions -p "..." | Raw | ~/.local/bin/agy (or `gemini` shim at ~/.claude/PAI/bin/gemini) |
| codex | codex exec "..." --sandbox read-only | Raw | ~/.local/bin/codex |
| ollama | bun Inference.ts --task PROFILE "..." | Raw (Inference.ts handles) | Uses Inference.ts |

### Output cleaning per engine:
- **Gemini**: Hook log lines appear after the response. Split on lines, drop from first hook-line.
- **All others**: Clean output natively.

### Subprocess wrapper (engine-invoke.ts):
Each engine invocation: Bun.spawn -> pipe stdout/stderr -> Promise.all on output + exit -> timeout with proc.kill -> return InvokeResult { stdout, stderr, exitCode, timedOut, elapsedMs }.

---

## 6. Security Model

- **Auth**: JWT Bearer token from OmniPulse PKI. Missing/invalid = 401.
- **Input validation**: skill limited to [a-zA-Z0-9_-], prompt capped at 64KB, timeout capped at 300s, caller limited to 64 chars.
- **Binding**: 127.0.0.1 only. No external network exposure.
- **Future hardening**: Per-caller rate limits, engine quotas, systemd IPAddressDeny.

---

## 7. Systemd Service

```ini
[Unit]
Description=PAI Skill Dispatcher Daemon
After=network.target

[Service]
Type=simple
ExecStart=/home/duane/.bun/bin/bun ~/.claude/PAI/Tools/SkillDispatcher/daemon.ts
Restart=always
RestartSec=5
Environment=HOME=/home/duane
Environment=PATH=%h/.bun/bin:%h/.local/bin:/usr/local/bin:/usr/bin:/bin
Environment=SKILL_DISPATCHER_PORT=31340

[Install]
WantedBy=default.target
```

---

## 8. File Layout

```
PAI/Tools/SkillDispatcher/
├── daemon.ts                  # Main HTTP server entrypoint
├── engine-invoke.ts           # Per-engine subprocess wrappers
├── skill-registry.json        # Skill -> engine mapping
├── auth.ts                    # JWT validation middleware
├── rate-limiter.ts            # Per-caller tracking (future)
├── types.ts                   # Shared TypeScript types
├── test-daemon.sh             # Smoke tests via curl
└── README.md                  # Quickstart
```

Files to modify for integration:
- PAI/Tools/OmniPulse/adapters/discord.ts — migrate route_to_engine to daemon HTTP call
- PAI/Tools/inference-matrix.json — add daemon route option
- ~/.config/systemd/user/ — new service file

---

## 9. Build Phases

| Phase | Deliverable | Effort |
|-------|------------|--------|
| #13 — Daemon skeleton | daemon.ts with /health + /engines, JWT auth, systemd service | ~30 min |
| #11 — Skill dispatch | skill-registry.json, engine-invoke.ts, /run_skill endpoint | ~45 min |
| #12 — Agent dispatch | /spawn_agent endpoint, agent registry, session tracking | ~30 min |
| Integration | discord.ts migration, inference-matrix update, ARCHITECTURE_SUMMARY | ~20 min |

---

## 10. Success Metrics

- Uptime: 99.9% (systemd watchdog)
- Latency overhead: <50ms vs direct subprocess
- Skill coverage: >=20 skills in registry
- Adapter coverage: >=3 adapters using daemon
- Error rate: <1% of requests

---

## 11. Open Design Questions

1. **Ollama A2A bridge**: Should the daemon expose an A2A endpoint for native Ollama agent calls? Currently OllamaSkill handles this indirectly.

2. **Streaming responses**: Should /run_skill support SSE for long tasks? Helpful for Discord slash commands that defer then PATCH.

3. **Pre-flight skill validation**: Should the daemon check that the requested skill exists before invoking the engine? Better errors vs. slightly more latency.

4. **Result caching**: Cache skill+prompt results? Risky (stale) but saves tokens for classification/summarization tasks.

5. **Agent isolation**: Worktree isolation for agent spawns? Daemon has no git concept — needs to be caller-driven.
