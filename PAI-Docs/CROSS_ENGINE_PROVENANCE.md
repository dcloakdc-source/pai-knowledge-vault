# Cross-Engine Provenance Convention

Establishes a shared append-only provenance log so every engine-to-engine delegation leaves a traceable record.

## Architecture

```
 Write (any engine)         Read (any engine)
       |                          |
       v                          v
 ┌─────────────┐          ┌─────────────┐
 │  provenance  │  ──sync──▶     ICM     │  fast semantic recall
 │  .jsonl      │  (future) │  (Qdrant)   │  via pulse_memory
 │  (source of  │          └─────────────┘
 │   truth)     │
 └─────────────┘
       │
       v
  git-tracked (optional)
  append-only, greppable
  reconstruct after disaster
```

- **JSONL** is the source of truth — append-only, atomic writes, auditable.
- **ICM** is the query layer — semantic search, cross-engine `pulse_memory`.
- **JSONL is NOT a cache.** It's the durable write-ahead log. ICM serves reads.

## The Problem

When engine A (e.g. PNC) is launched from engine B (e.g. PNK) — either manually or via `opencode run` / `Task` / direct CLI — there is no record of:
- Which engine originated the work
- Which session ID it came from
- What task was being delegated
- Where the output landed

This makes retrospective tracing impossible and leaves deliverables stranded in temp space.

## Registry Location

**Write target** — shared append-only JSONL at the runtime root:

```
~/MEMORY/provenance.jsonl
```

This path is accessible from all engines (home directory, cross-engine filesystem).

**Why runtime root, not `~/.claude/PAI/`:** Provenance is operational runtime data, not PAI content. The Ph2 memory-root migration established `~/MEMORY/` for this class of data.

## Record Format

Append-only JSONL — one JSON object per line, each with a `handoff_id` and `type` field:

```
{"handoff_id":"ho_20260622T044830_PNK_yolo-compare","type":"handoff","origin":{"engine":"PNK",...},"destination":{...},"task":{...},"deliverables":[...],"status":"completed","completed_at":"...","notes":"..."}
{"handoff_id":"ho_20260622T060000_PNC_something-else","type":"handoff","origin":{"engine":"PNC",...},"destination":{...},"task":{...},"status":"launched"}
```

### Handoff Record Schema

| Field | Required | Description |
|-------|----------|-------------|
| `handoff_id` | yes | Unique ID: `ho_<timestamp>_<origin-engine>_<slug>` |
| `type` | yes | Always `"handoff"` (future-proofing for record types) |
| `origin.engine` | yes | Engine that requested the work (PNK, PNC, PNX, PNG) |
| `origin.session_id` | yes | Session ID or path in the origin engine |
| `origin.timestamp` | yes | When the request was made |
| `origin.context` | no | Free-text context (e.g. "OpenCode workspace", "PAI Algorithm run") |
| `destination.engine` | yes | Engine that executed the work |
| `destination.session_id` | yes | Session ID or work directory path |
| `destination.project` | no | Project name (e.g. "-tmp", "-home-duane--claude") |
| `destination.timestamp` | yes | When the destination engine started |
| `task.summary` | yes | One-line task description |
| `task.skill_used` | no | Skill loaded to perform the work |
| `task.subagents` | no | Array of subagents spawned (type, toolcalls, target) |
| `deliverables[]` | no | Array of output artifacts (path, size, description) |
| `status` | yes | `launched` | `in_progress` | `completed` | `failed` |
| `completed_at` | no | When work finished |
| `notes` | no | Free text caveats or migration notes |

### Example Record

```json
{
  "handoff_id": "ho_20260622T044830_PNK_yolo-compare",
  "type": "handoff",
  "origin": {
    "engine": "PNK",
    "session_id": "opencode-session-uuid-or-path",
    "timestamp": "2026-06-22T04:48:30.000Z",
    "context": "OpenCode workspace in pai-opencode-local repo"
  },
  "destination": {
    "engine": "PNC",
    "session_id": "2026-06-22T04-48-30_prepare-a-comparative-comprehensive-security-risk",
    "project": "-tmp",
    "timestamp": "2026-06-22T04:48:30.873Z"
  },
  "task": {
    "summary": "Prepare comparative security risk assessment against Ultralytics YOLO and Meituan YOLO",
    "skill_used": "InfoSecRiskAssessment",
    "subagents": [
      { "type": "Explore", "engine": "PNC", "toolcalls": 29, "target": "Ultralytics YOLO" },
      { "type": "Explore", "engine": "PNC", "toolcalls": 16, "target": "Meituan YOLOv6" }
    ]
  },
  "deliverables": [
    {
      "path": "~/.claude/PAI/MEMORY/WORK/2026-06/2026-06-22T04-48-30_prepare-a-comparative-comprehensive-security-risk/deliverables/YOLO_Comparative_Security_Assessment.md",
      "size": 45466,
      "description": "Full STRIDE assessment"
    },
    {
      "path": "~/.claude/PAI/MEMORY/WORK/2026-06/2026-06-22T04-48-30_prepare-a-comparative-comprehensive-security-risk/deliverables/YOLO_Executive_Summary.md",
      "size": 9274,
      "description": "Decision brief"
    },
    {
      "path": "~/.claude/PAI/MEMORY/WORK/2026-06/2026-06-22T04-48-30_prepare-a-comparative-comprehensive-security-risk/deliverables/YOLO_Security_Hardening_Checklist.md",
      "size": 14049,
      "description": "Deployment hardening guide"
    }
  ],
  "status": "completed",
  "completed_at": "2026-06-22T04:52:05.675Z",
  "notes": "Output was initially written to /tmp/opencode/ — moved to permanent storage 2026-06-22"
}
```

## Write Protocol — RecordRouter (Recommended)

Use RecordRouter for all writes:

```bash
# Write handoff record (type: provenance)
bun ~/.claude/PAI/Tools/RecordRouter.ts record provenance '{"handoff_id":"ho_...","origin":{"engine":"PNC"},"destination":{"engine":"PNK"},"task":{"summary":"..."},"status":"launched"}'

# Mark handoff complete
bun ~/.claude/PAI/Tools/RecordRouter.ts complete ho_123 completed
bun ~/.claude/PAI/Tools/RecordRouter.ts complete ho_123 failed "Timeout"
```

RecordRouter handles: JSONL append (source of truth) + ICM sync (query layer) + nested metadata (`_meta.engine`, `_meta.recorded_at`).

**Raw JSONL append** (fallback, no ICM sync):
```bash
echo '{"handoff_id":"ho_...","type":"handoff",...}' >> ~/MEMORY/provenance.jsonl
```

**Anti-patterns:**
- ❌ Overwriting or editing existing records. JSONL is append-only. Use `complete` for status transitions.
- ❌ Per-handoff files in a directory. That's what was replaced.
- ❌ Embedding provenance in git commit messages only — not machine-queryable.

## Read / Discovery — RecordRouter (Recommended)

```bash
# Unified query across ICM + JSONL
bun ~/.claude/PAI/Tools/RecordRouter.ts query "yolo security"
bun ~/.claude/PAI/Tools/RecordRouter.ts query --topic provenance "handoff"

# Session-start check for incoming handoffs
bun ~/.claude/PAI/Tools/RecordRouter.ts check

# List only incoming (non-completed) handoffs
bun ~/.claude/PAI/Tools/RecordRouter.ts handoffs 10

# All routes
bun ~/.claude/PAI/Tools/RecordRouter.ts routes
```

**Via ICM directly (for ad-hoc queries):**
```bash
# Recall with semantic search
icm recall "cross-engine handoff yolo" --limit 5

# List by topic
icm list --topic provenance-provenance-PNK

# Forget (cleanup test entries)
icm forget <entry-id>
```

**Via JSONL (for grep/tail scripting):**
```bash
tail -5 ~/MEMORY/provenance.jsonl | jq .
grep '"PNC"' ~/MEMORY/provenance.jsonl | jq .
grep -c '"status": "completed"' ~/MEMORY/provenance.jsonl
```

## ICM Integration (Live)

**Status:** LIVE as of 2026-06-23.

Every `RecordRouter record` and `RecordRouter complete` call automatically syncs to ICM under topic `provenance-<type>-<engine>` (launches) or `provenance-completion-<engine>` (completions). The `icm` CLI at `~/.local/bin/icm` is available from all engines (PNC, PNK, PNX, PNG) — confirmed working from PNK.

The layer model:
```
JSONL  ──sync──▶  ICM (Qdrant)
(source of truth)  (semantic query layer)
```

JSONL remains the append-only source of truth. ICM is the fast-read semantic index. Both are always in sync (atomic JSONL write → sync to ICM).

## Engine-Specific Integration Points

### PNK (OpenCode) — AGENTS.md § OBSERVE Phase
- Session start: `RecordRouter.ts check` → surfaces incoming handoffs
- Task completion: `RecordRouter.ts complete <id>` → closes the loop

### PNC (Claude Code) — SessionStart.hook.ts
- Session start: queries RecordRouter for incoming handoffs
- If found, injects `🔄 RELAY RECOVERY` block into additionalContext
- PNC sees the same handoff data PNK does

### PNC (Claude Code) — AgentStart.hook.ts
- On every Task subagent spawn, detects cross-engine targets (PNK/PNX/PNG/OpenCode/Codex/Gemini)
- Auto-records `provenance` entry with `status: launched`
- No overhead for non-cross-engine spawns

### Completion Loop
1. **PNC spawns** → AgentStart writes `launched` record
2. **Destination engine discovers** → check/query shows incoming handoff
3. **Destination engine completes** → `RecordRouter complete` writes completion
4. **PNC next session** → SessionStart shows completed handoff (via relay recovery, only shows non-completed)

---

*Established 2026-06-22. Trigger incident: YOLO assessment produced by PNC from PNK context with no trace until manual recovery from /tmp/opencode/.*
*Rewrite 2026-06-22: Moved from per-handoff flat files to append-only JSONL + ICM architecture.*
*Live pipeline 2026-06-23: RecordRouter (CLI + MCP), OBSERVE integration (PNK AGENTS.md + PNC SessionStart hook), auto-recording (PNC AgentStart hook), complete loop (`RecordRouter complete` command), overlay schema (routing.json v1.2.0). Provenance log: 4 permanent entries across PNK→PNC, PNK→PNG, PNC→PNK.*
