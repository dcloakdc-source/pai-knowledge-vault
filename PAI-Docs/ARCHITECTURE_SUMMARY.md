# PAI Architecture Summary

**PAI Nova v5.0.0 on pai-primary (Linux)**

> Auto-generated summary for @ import — see full docs in PAI/DOCUMENTATION/ for detail.

---

## System Overview

PAI is a Life Operating System running on pai-primary (Proxmox homelab, Ubuntu). The DA is **PAI Nova**. Three AI engines operate under this single DA identity:

| Engine | Interface | Primary Model |
|--------|-----------|--------------|
| Claude Code | Terminal, IDE | claude-sonnet-4-6 |
| Antigravity | Google VS Code fork | Gemini Flash |
| Ollama | Local inference | llama3.2:3b / gemma2:9b / qwen2.5:7b |

---

## Core Services (systemd user units)

| Service | Port | Purpose |
|---------|------|---------|
| pai-voice-server | 8888 | Chatterbox TTS (voice notifications) |
| omnipulse | 31338 | **OmniPulse gateway** (unified daemon — hooks, health, dashboard, agents) |
| pai-pulse | 31337 | **DEPRECATED** — legacy Pulse daemon (use OmniPulse :31338) |
| pai-memory | varies | MCP memory server |
| pai-command-center | varies | Command center UI (proxied via OmniPulse) |
| pai-entity-graph | varies | Entity graph service |
| pai-cc | varies | Claude Code relay |

## HA / Redundancy Baseline

PAI currently has restart and backup resilience, not automatic failover. The canonical HA discussion is `PAI/INFRA/HA_REDUNDANCY.md`, backed by `PAI/Tools/HA/topology.json` and the local probe:

```bash
bash ~/.claude/PAI/Tools/HA/ha-status.sh --summary
```

Latest baseline from 2026-06-08: Tier 1 services were healthy (OmniPulse, Functions API, Command Center, Ollama), 8/9 tracked services were active, and NAS backup heartbeat was present. Next architecture step: prepare M710q/prox warm standby with non-secret service bundle, restore sequence, smoke tests, and manual promotion checklist.

---

## Memory Architecture (5 layers)

1. **Claude Code native** — `~/.claude/projects/` (session transcripts)
2. **ICM MCP** — Infinite Context Memory (2 MCP servers: `icm`, `pai-memory`)
3. **Learning files** — `MEMORY/LEARNING/` (3820 files, reflections, patterns)
4. **Entity graph** — 1936 entities / 93K edges (Substrate + custom)
5. **Relay system** — Cross-session handoff at 85%/93% context thresholds

---

## Key Paths

```
~/.claude/                      PAI root
~/.claude/PAI/                  PAI core (Algorithm, Tools, USER, PULSE, DOCUMENTATION)
~/.claude/PAI/USER/             Identity + TELOS + preferences
~/.claude/PAI/PULSE/            Pulse daemon
~/.claude/PAI/Algorithm/        Algorithm versions (<!-- SPINE:GENERATED:algorithm_version -->v5.7.5<!-- /SPINE:GENERATED:algorithm_version --> current)
~/.claude/MEMORY/               Work, state, observability, learning
~/.claude/skills/               74 custom + public skills
~/.claude/openspec/changes/     13 OpenSpec proposals in flight
~/.config/PAI/secrets.env       API keys and credentials
```

---

## Model Routing Infrastructure

- **ModelMatrix** (`PAI/Tools/ModelMatrix/`) — 20 models cataloged with capability tags, cost, context windows
- **Inference dispatcher** (`PAI/Tools/Inference.ts`) — Routes 13 task types to models across Ollama/Claude/Gemini/OpenAI
- **Routing matrix** (`PAI/Tools/inference-matrix.json`) — Task→Model mappings with primary/fallback
- **Model Benchmark System** (`PAI/DOCUMENTATION/MODEL_BENCHMARK_SYSTEM.md`) — Quality-calibrated routing design (Phase 1 imminent)
- **SettingsLab** (`~/.claude/skills/SettingsLab/`) — Self-improving settings with harvest→compare→refine→retest→apply loop
- **Evals** (`~/.opencode/skills/Utilities/Evals/`) — Agent evaluation framework with CompareModels workflow

## Org RBAC & Archetype Engagement

- **RBAC org model** (`PAI/RBAC_ORG_STRUCTURES_MODEL.md`) — Defines scoped org roles and authority archetypes: Principal, Coordinator, Producer, Verifier, Operator, Security, Archivist, Broker, Caregiver.
- **Engagement policy** (`PAI/config/archetype-engagement.example.yml`) — Maps task keywords and context signals to recommended archetypes before ORA assignments exist.
- **Engagement classifier** (`PAI/Tools/validate-archetype-engagement.py`) — Read-only dry-run classifier; emits recommended archetypes, default org structure, principal-required status, preferred candidates, and trace.
- **Design/UI/UX/art routing** — Treated as capability routing under Producer + Verifier, not a separate authority archetype.

## Active Infrastructure Projects

| Project | Status | Description |
|---------|--------|-------------|
| PAI v5.0.0 staged upgrade | IN PROGRESS | This migration |
| Model benchmark calibration | DESIGN | Per-task quality scores replacing static matrix |
| RLSD training pipeline | ACTIVE | rlsd_etl.py + train_rlsd.py + benchmark |
| OpenSpec proposals | 13 NEEDS REVISION | Phase 0 review complete, builds queued |
| Sovereign Identity | PENDING Phase 2a | DB isolation, enrollment pending |
| Proxmox Windows recovery | PENDING | nvme1n1 data recovery |

---

*Full documentation: PAI/DOCUMENTATION/ — MODEL_BENCHMARK_SYSTEM.md (design), AlgorithmSystem.md, MemorySystem.md, SkillSystem.md, HookSystem.md, PulseSystem.md*
