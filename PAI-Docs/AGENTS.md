# PAI Agents Directory

This file documents the external-agent surface PAI exposes — the entry points
through which non-Claude-Code agents (LangGraph, A2A peers, MCP clients,
custom tools) interact with PAI memory, skills, and Algorithm capabilities.

**Ownership:** `pai-mcp-server` proposal owns this file (resolved D-07,
2026-05-03). The `a2a-agent-card-expansion` proposal contributes a sub-section
but does not own the document.

## 1. PAI MCP Server (`pai-mcp-server`)

**Endpoint:** stdio JSON-RPC (MCP protocol). Spawned by Claude Code as a
configured MCP server, OR invoked directly: `bun ~/.claude/PAI/Tools/pai-mcp-server.ts`.

### Built-in tools

| Tool | Purpose | ACL tier required |
|------|---------|-------------------|
| `pai_icm_search` | Search PAI ICM long-term memory | any (read-only) |
| `pai_icm_store` | Stage or write a memory entry | tier-aware (see §2) |

### Dynamic skill exposure

The MCP server scans `~/.claude/skills/<name>/skill.schema.json` and exposes
every entry whose `mcp_exposed: true` flag is set. Each becomes an MCP tool
named `skill/<name>`.

Schemas with `executable:` (and a non-truncated allowlisted binary prefix)
are invoked via `runSkillExecutable()` — `execFile` with argv array, no
shell. Schemas with `requires_claude: true` are invoked via
`runSkillJudgment()` — `claude -p` with prompt frame + ARGS_JSON via stdin.

Allowlist: `bun, node, python3, claude, curl, echo, jq, rg, openspec,
pi-gsd-tools, git, gh, yt-dlp, date, ls, cat, grep`. Anything else is
rejected as schema misconfiguration or attack.

## 2. Memory ACL (tier model)

External agents writing to ICM go through `MEMORY/STATE/pending-memories.jsonl`,
not directly to the SQLite store. SessionStart's `PromotePending` hook decides
per-entry: auto-promote or surface-for-review.

| Tier | Determination | Write behavior |
|------|---------------|----------------|
| `trusted` | Bearer token from `~/.config/PAI/pki/credentials/memory-acl.token` matches | Direct ICM write with `source_agent:` / `source_tier:` content prefix |
| `external` | Default for any caller with no token | Stage to pending; novelty + conflict score; auto-promote if score > 0.8 AND no conflicts AND not novel |
| `readonly` | Explicit `settings.json memory_acl.agents[<id>] = "readonly"` | 403 reject — no write reaches pending file |

**Security guarantees:**
- Origin alone NEVER grants `trusted` (token required)
- Novel-topic entries (no ICM hits ≥ 0.5 score) forced to `auto_promote_score ≤ 0.4` regardless of importance — defends against malicious novel-fact injection

## 3. Observability (event emission)

Every MCP tool call passes through PAI's hook pipeline and emits structured
events to `MEMORY/STATE/hook-events.jsonl`:

- `tool_start` / `tool_end` — every tool call
- `agent_handoff` — when Task/Agent tool invoked
- `memory_promoted` / `memory_rejected` — pending memory decisions
- `session_start` / `session_end` / `agent_end` — lifecycle

Schema: `PAI/Schema/hook-events.ts`. Validator (`validatePAIHookEvent`)
runs on every emit before append, and on every `POST /events` ingest at the
local SSE server (`PAI/Tools/ObservabilityServer.ts` on `127.0.0.1:8895`,
gated by `PAI_OBSERVABILITY_SSE=1`).

Optional OTel export: set `OTEL_EXPORTER_OTLP_ENDPOINT` to forward spans
to an external collector. Zero cost when unset.

## 4. A2A Agent Card (sub-section — owned by `a2a-agent-card-expansion`)

> **Proposal status:** NEEDS REVISION (Phase 3). This sub-section is a
> placeholder; canonical content lives in the proposal at
> `~/.claude/openspec/changes/a2a-agent-card-expansion/proposal.md`.

When A2A integration ships, agents discovering PAI via agent-card protocol
will see:
- The MCP tool list above (mirrored as A2A capabilities)
- The tier model (§2) — same auth contract
- Event emission guarantees (§3) — same observability surface
- A2A-specific skill-invoke endpoint with the same `runSkillExecutable` /
  `runSkillJudgment` security contract

Until A2A ships, external agents use the MCP path.

## 5. Authentication

| Channel | Auth mechanism | Token source |
|---------|----------------|--------------|
| MCP (stdio) | Bearer token via `params._auth` in JSON-RPC | `~/.config/PAI/pki/credentials/memory-acl.token` |
| SSE GET `/events` | Origin allowlist (no token; loopback only) | n/a |
| SSE POST `/events` | Origin allowlist + schema validation | n/a (loopback only) |
| OTel export | None — outbound only, opt-in via env var | n/a |

External agents calling without a token are tier `external` (read OK,
write goes to pending). External agents on a non-loopback Origin (when
SSE is enabled) get `403 Forbidden` regardless of token.

## See also

- Memory ACL implementation: `PAI/Tools/MemoryACL.ts`
- MCP server implementation: `PAI/Tools/pai-mcp-server.ts`
- Skill execution helper: `PAI/Tools/skill-exec.ts`
- Schema source of truth: `PAI/Schema/hook-events.ts`
- Pending memory promote hook: `hooks/PromotePending.hook.ts`
- SSE / OTel: `PAI/Tools/ObservabilityServer.ts`, `PAI/Tools/OTelExporter.ts`

<!-- switchboard:agents-protocol:start -->
# AGENTS.md - Switchboard Protocol

## 🚨 STRICT PROTOCOL ENFORCEMENT 🚨

This project relies on **Switchboard Workflows** defined in `.agent/workflows`.

**Rule #1**: If a user request matches a known workflow trigger, you **MUST** execute that workflow exactly as defined in the corresponding `.md` file. Do not "wing it" or use internal capability unless explicitly told to ignore the workflow.

**Rule #2**: You MUST NOT call `send_message` with unsupported actions. Only `submit_result` and `status_update` are valid (see Code-Level Enforcement below). The tool will reject unrecognized or unauthorized actions.

**Rule #3**: The `send_message` tool auto-routes actions to the correct recipient based on the active workflow. You do NOT need to specify a recipient. If the workflow requires a specific role (e.g. `reviewer`), ensure an agent with that role is registered.

### Workflow Registry

| Trigger Words | Workflow File | Description |
| :--- | :--- | :--- |
| `/accuracy` | **`accuracy.md`** | High accuracy mode with self-review (Standard Protocol). |
| `/improve-plan` | **`improve-plan.md`** | Deep planning, dependency checks, and adversarial review. |
| `/challenge`, `/challenge --self` | **`challenge.md`** | Internal adversarial review workflow (no delegation). |
| `/chat` | **`chat.md`** | Activate chat consultation workflow. |
| `/archive` | **`archive.md`** | Query or search the plan archive. |
| `/export` | **`export.md`** | Export current conversation to archive. |


### ⚠️ MANDATORY PRE-FLIGHT CHECK

Before EVERY response, you MUST:

1. **Scan** the user's message for explicit workflow commands from the table above (prefer `/workflow` forms).
2. **Do not auto-trigger on generic language** (for example: "review this", "delegate this", "quick start") unless the user explicitly asks to run that workflow.
3. **If a command match is found**: Read the workflow file with `view_file .agent/workflows/[WORKFLOW].md` and execute it step-by-step. Do NOT improvise an alternative approach.
4. **Fast Kanban Resolution**: If the user asks about plans in specific Kanban columns (e.g. "update all created plans"), you MUST use the `get_kanban_state` MCP tool to instantly identify the target plans.
5. **If no match is found**: Respond normally.

### Execution Rules

1. **Read Definition**: Use `view_file .agent/workflows/[WORKFLOW].md` to read the steps.
2. **Execute Step-by-Step**: Follow the numbered steps in the workflow.
   - If a step says "Call tool X", call it.
   - If a step says "Generate artifact Y", generate it.
3. **Do Not Skip**: Do not merge steps or skip persona adoption unless the workflow explicitly allows it (e.g. `// turbo`).
4. **Do Not Improvise**: If a workflow exists for the user's request, you MUST use it. Calling tools directly without following the workflow is a protocol violation and will be rejected by the tool layer.

### Code-Level Enforcement

The following actions are enforced at the tool level and WILL be rejected if misused:

| Action | Required Active Workflow |
| :--- | :--- |
| `submit_result` | *(no restriction — this is a response)* |
| `status_update` | *(no restriction — informational)* |

Sending to non-existent recipients is always rejected (even when auto-routed).

### 🏗️ Switchboard Global Architecture

```
User ──► Switchboard Operator (chat.md)
              │  Plans captured in .switchboard/plans/
              │
              ├──► /improve-plan   Deep planning, dependency checks, and adversarial review
              └──► Kanban Board    Plans moved through workflow stages (Created → Coded → Reviewed → Done)

All file writes to .switchboard/ MUST use IsArtifact: false.
Plans are executed via Kanban board workflow, not delegation.
```

Conversational routing: when the intent is to advance a kanban card or send a plan to the next agent/stage, prefer `move_kanban_card(sessionId, target)` over raw `send_message`. The `target` may be a kanban column label, a built-in role, or a kanban-enabled custom agent name; generic conversational `coded` / `team` targets are smart-routed by plan complexity.

### 📚 Available Skills

Skills provide specialized capabilities and domain knowledge. Invoke with `skill: "<name>"`.

| Skill | When to Use |
|-------|-------------|
| `archive` | User asks to "search archives", "query archives", "find old plans", "export conversation" |
| `review` | User asks to review code changes, a PR, or specific files |

**Usage**: Call `skill: "archive"` before performing archive operations to access detailed tool documentation and examples.

**Skill Files Location**: `.agent/skills/` (distributed with plugin)
<!-- switchboard:agents-protocol:end -->
