# Rearchitecture Gap Map

> Ph2 shipped filesystem layout consolidation (`~/.claude/PAI/MEMORY/` content
> layout, `~/MEMORY/` runtime state) but **centralized memory in ICM never
> shipped**. These 9 gaps correct that: layered architecture with JSONL source
> of truth + ICM semantic index + flat-file per-engine overlays.

## Overview

The original Ph2 rearchitecture concept was a single monolithic ICM store for
all memory types. What actually shipped was only the filesystem layout
consolidation. This gap map captures the **correction** — a layered approach
where data type determines storage, and ICM is the query layer, not the
single store.

## Gap Map

| Tier | # | Gap | Status | Depends On | Owner |
|------|---|-----|--------|------------|-------|
| **0** | 1 | Cross-engine session ID resolution | ✅ `session-id.ts` — all 4 engines mapped | — | PNK |
| **0** | 2 | Centralized provenance tracking | ✅ RecordRouter + `provenance.jsonl` + ICM sync | #1 | PNK |
| **0** | 3 | Engine-sourced overlay configuration | 🟡 Stub in routing.json (schema TBD) | — | PNK |
| **1** | 4 | ICM write path from all engines | ✅ `icm` CLI at `~/.local/bin/icm` — confirmed all engines | — | PNK |
| **1** | 5 | Unified cross-engine query | ✅ RecordRouter `query` — searches ICM + all JSONL logs | #4 | PNK |
| **1** | 6 | PAI_ENGINE env var for all engines | 🟡 PNK set in MCP env, PNC harnessed, PNX/PNG need harness setup | — | PNK/PNX/PNG |
| **2** | 7 | PAI_DIR extraction scope | 📋 Scope doc needed — 165 callers to map | — | PNK |
| **3** | 8 | WS-7 cutover sequencing | 📋 Sequencing plan needed — phased legacy ref migration | #7 | PNK |
| **4** | 9 | Secrets management plan | 📋 Plan needed — vault-backed secrets for RESTIC_PASSWORD et al. | — | PNK |

## Status Key

| Status | Meaning |
|--------|---------|
| ✅ Done | Implemented and verified |
| 🟡 Partial | Implementation exists but not complete |
| 📋 Planned | Scope/Spec/Plan doc needed |
| — | Not started |

## Architecture: Layered, Not Monolithic

```
┌─────────────────────────────────────────────┐
│            RecordRouter (dispatcher)          │
├────────────┬──────────┬──────────┬───────────┤
│ Provenance │ Decision │ Note     │ State     │
│ → JSONL    │ → JSONL  │ → JSONL  │ → JSONL   │
│ + ICM sync │ + ICM    │ + ICM    │ + ICM     │
├────────────┴──────────┴──────────┴───────────┤
│          ICM Semantic Index (query layer)     │
├──────────────────────────────────────────────┤
│     Flat Files (per-engine overlay state)     │
└──────────────────────────────────────────────┘
```

- **JSONL**: Append-only source of truth for records (provenance, decisions, notes).
  Greppable, replayable, reconstructable.
- **ICM**: Semantic query layer. Stores indexed copies for recall/search.
  Source of truth for unstructured context entries.
- **Flat files**: Per-engine state overlays (`.opencode/`, `.codex/`, `.gemini/`).

## Dependencies Between Gaps

```
#1 (Session IDs) → #2 (Provenance) ─┐
                                     ├──→ #5 (Unified Query)
#4 (ICM Write) ─────────────────────┘
                                          → #7 (PAI_DIR) → #8 (Cutover)
                                          → #9 (Secrets) (independent)
```

Gaps #1, #2, #4, #5 are the critical path for cross-engine memory coordination.
Gaps #7-#9 are the PAI_DIR home-migration workstream, independent of memory.
