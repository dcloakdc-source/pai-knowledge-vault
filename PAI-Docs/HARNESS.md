# PAI Harness

**Version:** 4.0.0 — last updated 2026-04-23
**Principal:** Duane
**Primary DA:** Nova (AGT-001 / PNC — Claude Code)
**Algorithm:** <!-- SPINE:GENERATED:algorithm_version -->v5.7.5<!-- /SPINE:GENERATED:algorithm_version -->

This document is the master reference for the PAI harness: the infrastructure layer that configures, constrains, and coordinates all AI assistants operating in the PAI ecosystem. Read this file to understand how PAI works end-to-end, and see the [Per-Agent Alignment Guide](#per-agent-alignment-guide) for what your specific agent type must do.

For deeper reading on any subsystem, see the [Context Routing Table](CONTEXT_ROUTING.md).

---

## What Is the Harness

The harness is the set of configuration, hooks, and conventions that shape how AI assistants behave. It is not a single file — it is a layered system across six levels, from environment configuration at the bottom to multi-agent coordination at the top. The harness ensures that assistants:

1. Operate with a consistent identity and behavioral contract
2. Execute work through a structured, verifiable method (The Algorithm)
3. Persist state and learning across sessions
4. Coordinate with other agents using shared protocols
5. Maintain security, quality, and token-efficiency standards

**Only Claude Code (AGT-001 PNC) has full harness integration.** Gemini CLI, OpenCode, and Ollama models have partial integration via shared-context files and delegation protocols — but they must still follow [Universal Behavioral Standards](#universal-behavioral-standards).

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│  Layer 6: Multi-Agent Coordination                          │
│  Shared-Inbox / Debate / Switchboard / Token Routing        │
├─────────────────────────────────────────────────────────────┤
│  Layer 5: Capabilities                                      │
│  <!-- SPINE:GENERATED:skill_count -->107<!-- /SPINE:GENERATED:skill_count --> Skills (Skill tool) + TypeScript Tools (PAI/Tools/)    │
├─────────────────────────────────────────────────────────────┤
│  Layer 4: Persistent Memory                                 │
│  MEMORY/WORK, LEARNING, STATE, WISDOM, RESEARCH             │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: Execution Engine                                  │
│  Algorithm v3.8.0 (7 phases) + PRDs + ISC methodology       │
├─────────────────────────────────────────────────────────────┤
│  Layer 2: Session Lifecycle                                 │
│  <!-- SPINE:GENERATED:hook_count -->104<!-- /SPINE:GENERATED:hook_count --> Hooks (SessionStart → UserPromptSubmit → … → Stop)     │
├─────────────────────────────────────────────────────────────┤
│  Layer 1: Identity Layer                                    │
│  settings.json → CLAUDE.md (generated) + AISTEERINGRULES    │
└─────────────────────────────────────────────────────────────┘
```

---

## Layer 1: Identity Layer

### settings.json — Single Source of Truth

`~/.claude/settings.json` is the single source of truth for all PAI configuration. Everything else is derived from it.

**Five key configuration areas:**

| Area | What It Controls | Key Fields |
|------|-----------------|------------|
| **Environment** | Paths, API endpoints, model routing | `PAI_DIR`, `OLLAMA_HOST`, `PROJECTS_DIR`, `ANTHROPIC_SMALL_FAST_MODEL` |
| **Identity** | Who the DA is, principal name | `env.principal`, DA name (in `daidentity` section) |
| **Permissions** | What Claude Code can/cannot do | `permissions.allow`, `permissions.deny`, `permissions.ask` |
| **Hooks** | Which lifecycle hooks run | `hooks` object — all hook registrations live here |
| **Startup Loading** | Files loaded at session start | `loadAtStartup.files`, `dynamicContext` toggles |

**Critical environment variables (always verify these paths first):**

```bash
PAI_DIR=/home/duane/.claude          # All PAI lives here
OLLAMA_HOST=http://192.168.50.20:11434  # Local LLM inference
PROJECTS_DIR=/PAI/                   # Additional working dirs
ANTHROPIC_SMALL_FAST_MODEL=ollama/llama3.2:3b  # Default small/fast routing
PAI_OLLAMA_FAILOVER=true             # Auto-failover to Ollama on budget pressure
```

### CLAUDE.md — The Master Config

`~/.claude/CLAUDE.md` is the instruction file that Claude Code loads at every session start. It defines the DA's identity, behavioral modes, the algorithm reference, and the context routing table.

**It is generated, not hand-edited.** The source is:
```bash
bun ~/.claude/PAI/Tools/BuildCLAUDE.ts
```
Which combines: `CLAUDE.md.template` + `settings.json` + `PAI/Algorithm/LATEST` pointer.

A `SessionStart` hook (`LoadContext.hook.ts`) rebuilds CLAUDE.md if the template is newer than the output, keeping it fresh automatically.

**Three execution modes defined in CLAUDE.md:**

| Mode | Trigger | Format |
|------|---------|--------|
| **MINIMAL** | Greetings, ratings, acknowledgments | Short `═══ PAI ═══` block |
| **NATIVE** | Single-step tasks under ~2 minutes | `════ PAI \| NATIVE MODE ═══` block |
| **ALGORITHM** | Everything else — complex, multi-step, multi-file | Full 7-phase Algorithm execution |

Every response must use exactly one mode. The mode header is the first visible output — no freeform text before it.

### AI Steering Rules

`PAI/AISTEERINGRULES.md` defines universal behavioral rules loaded at startup:
- Surgical fixes only — never gut components
- Never assert without verification via tool call
- Build ISC from every request before executing
- Ask before destructive actions
- Read before modifying
- One change when debugging
- Minimal scope — only what was asked

User-specific overrides live in `PAI/USER/AISTEERINGRULES.md`.

---

## Layer 2: Session Lifecycle (Hooks)

Hooks are TypeScript scripts in `~/.claude/hooks/` that run automatically at lifecycle events. They are defined in `settings.json → hooks` and run asynchronously (they enhance but never block Claude Code).

**Six hook event types:**

| Event | When | Examples |
|-------|------|---------|
| `SessionStart` | Session begins | LoadContext, CheckVersion, ConfigReload |
| `UserPromptSubmit` | User sends message | FormatReminder, ComplexityRouter, ElicitationGuard |
| `PreToolUse` | Before any tool call | SecurityValidator, BudgetGuard, DeferRiskyOps |
| `PostToolUse` | After tool call completes | PRDSync, AlgorithmTracker, ImplicitSentimentCapture |
| `Stop` | Claude finishes a response | ExplicitRatingCapture, WorkCompletionLearning |
| `SessionEnd` | Session terminates | ClaudeMdIntegrity, GenerateHandoff, AccountabilityCheck |

**Four critical hooks (understand these first):**

| Hook | Function |
|------|---------|
| `SecurityValidator.hook.ts` | Guards all Bash/Edit/Write/Read calls — path validation, injection prevention, secret scanning. Blocked calls require explicit override. |
| `BudgetGuard.hook.ts` | Tracks 7-day rolling token spend; routes to Ollama when approaching budget limits; injects routing recommendations. |
| `FormatReminder.hook.ts` | Detects when a response uses freeform text instead of a PAI mode format; injects a reminder to use the correct format. |
| `LoadContext.hook.ts` | Injects startup files (steering rules, user context, learning readback, relationship context) at session start. |

**See also:** `PAI/THEHOOKSYSTEM.md` for the full hook catalog with event types and configuration patterns.

---

## Layer 3: Execution Engine

### The Algorithm v3.8.0

The Algorithm is the 7-phase execution engine for all complex work. Source: `PAI/Algorithm/v3.8.0.md`. It transitions from **Current State → Ideal State** via verifiable criteria.

**Seven phases:**

| # | Phase | Entry Action | Default Model |
|---|-------|-------------|---------------|
| 1 | **OBSERVE** | Voice: "Entering the Observe phase." → PRD timestamp | haiku |
| 2 | **THINK** | Voice: "Entering the Think phase." → PRD phase update | sonnet |
| 3 | **PLAN** | Voice: "Entering the Plan phase." → PRD phase update | sonnet |
| 4 | **BUILD** | Voice: "Entering the Build phase." → PRD phase update | — |
| 5 | **EXECUTE** | Voice: "Entering the Execute phase." → PRD phase update | — |
| 6 | **VERIFY** | Voice: "Entering the Verify phase." → PRD phase update | haiku |
| 7 | **LEARN** | Voice: "Entering the Learn phase." → PRD to complete | — |

Voice announcements use: `curl -s -X POST http://localhost:31337/notify -H "Content-Type: application/json" -d '{"message": "...", "voice_id": "main", "voice_enabled": true}'`

**Only the primary agent (Claude Code PNC) executes voice announcements. Subagents skip all voice curls.**

### Effort Tiers

| Tier | Budget | ISC Floor | When |
|------|--------|-----------|------|
| Standard | <2 min | 8 | Normal request |
| Extended | <8 min | 16 | Quality must be extraordinary |
| Advanced | <16 min | 24 | Substantial multi-file work |
| Deep | <32 min | 40 | Complex design |
| Comprehensive | <120 min | 64 | No time pressure |

### ISC Methodology — Ideal State Criteria

Every Algorithm run generates ISC (Ideal State Criteria): binary-testable, atomic checkboxes that define "done."

**The Splitting Test (apply to every criterion before writing):**
1. Contains "and"/"with" joining two verifiable things? → split
2. Can part A pass while part B fails independently? → split
3. Contains "all"/"every"/"complete"? → enumerate what that means
4. Crosses domain boundaries (UI/API/data/logic)? → one criterion per boundary

ISC count must meet the effort tier floor. Below the floor = decompose and recount. No exceptions.

### PRDs — Product Requirements Documents

Every Algorithm run creates a PRD in `~/.claude/MEMORY/WORK/{slug}/PRD.md`.

**8 required frontmatter fields:**

```yaml
task: "Imperative description, max 60 chars"
slug: YYYYMMDD-HHMMSS_kebab-description
effort: standard|extended|advanced|deep|comprehensive
phase: observe|think|plan|build|execute|verify|learn|complete
progress: 0/8
mode: interactive|loop
started: 2026-04-23T11:43:47Z
updated: 2026-04-23T11:43:47Z
```

**4 body sections** (appear only when populated):
- `## Context` — what was requested, why it matters, constraints, risks
- `## Criteria` — ISC checkboxes (`- [ ] ISC-1: text`)
- `## Decisions` — timestamped non-obvious choices
- `## Verification` — evidence per criterion (tool-call output proving each ISC passed)

The AI is the sole writer of PRDs. Hooks (PRDSync) only read PRDs to sync state to `work.json`. See `PAI/PRDFORMAT.md` for full spec.

---

## Layer 4: Persistent Memory

`~/.claude/MEMORY/` stores all cross-session state.

**Five primary directories:**

| Directory | Contents | Who Writes |
|-----------|----------|-----------|
| `WORK/` | PRDs — one per Algorithm session, indexed by slug | AI (during Algorithm) |
| `LEARNING/` | Algorithm reflections, failure patterns, signals, synthesis | Hooks + AI (LEARN phase) |
| `STATE/` | token-routing-matrix, work.json, security-overrides, session names | Hooks + AI |
| `WISDOM/` | Domain knowledge frames (`development`, `security`, `architecture`, etc.) | AI (on upgrade) |
| `RESEARCH/` | Research agent output captures | Research hooks |

**Memory access during Algorithm:**
- OBSERVE: Read `MEMORY/STATE/token-routing-matrix.md` for budget routing
- OBSERVE: Wisdom injection from `MEMORY/WISDOM/FRAMES/{domain}.md`
- LEARN: Append to `MEMORY/LEARNING/REFLECTIONS/algorithm-reflections.jsonl`

ICM (Infinite Context Memory) MCP server provides semantic search across memory: `mcp__icm__icm_memory_recall`.

---

## Layer 5: Capabilities

### Skills System

Skills are self-contained capability packages in `~/.claude/skills/`, each with a `SKILL.md` that defines triggers, workflows, and required tools. There are 52+ skills across 12 categories.

**Invocation pattern:**
```
Skill("ResearchName")          # For PAI skills
Task({ subagent_type: "..." }) # For Claude Code built-in agent types
```

**The Phantom Prevention Rule (critical):** Selecting a capability in OBSERVE creates a binding obligation to invoke it via `Skill` or `Task` tool call in BUILD/EXECUTE. Writing output that resembles a skill's output without calling the tool is dishonest and a critical failure. If a skill is selected but not needed, remove it from the list with a reason.

**Effort-scaled minimum invocations:**
- Standard: 1-2 capabilities
- Extended: 3-5
- Advanced: 4-7
- Deep/Comprehensive: 6-15

See the system-reminder at session start for the full skills listing.

### TypeScript Tools

`PAI/Tools/` contains utility scripts:
- `BuildCLAUDE.ts` — regenerate CLAUDE.md from template
- `Inference.ts` — AI calls (use this instead of importing Anthropic SDK directly)
- `BudgetReport.ts` — token spend report
- `ReflectionReview.ts` — surface recurring failure patterns
- `WisdomDomainClassifier.ts` — identify relevant wisdom domains for a task

---

## Layer 6: Multi-Agent Coordination

### Agent Registry

| ID | Name | Type | Role |
|----|------|------|------|
| AGT-001 | PNC | Claude Code | Primary DA — deep reasoning, architectural orchestration, Algorithm execution |
| AGT-002 | PNG | Antigravity CLI (`agy`) | Research, web search, factual lookup, large-context analysis |
| AGT-003 | opai | OpenCode | Autonomous coding, legacy cleanup, batch coding tasks |
| AGT-004 | Switchboard-Broker | Service | Real-time message routing between agents |
| AGT-005 | Antigravity IDE | Antigravity IDE (VS Code fork) | Visual Gemini workspace, UI/UX, research board, planning artifacts, brain artifacts |

### Shared-Inbox Protocol

`PAI/Shared-Inbox/` is the inter-agent communication directory:

```
PAI/Shared-Inbox/
├── shared-context/          # Context cards for non-CC agents (read at their session start)
│   ├── _PAI_Capabilities.md # What Claude Code can do — read by Antigravity/PNG
│   └── _Model_Routing.md    # When to use Claude vs Gemini — read by all agents
├── debate/                  # Debate protocol artifacts (Claude ↔ Gemini)
│   └── {id}/
│       ├── claude_position.md
│       ├── gemini_position.md
│       └── synthesis.md
├── to-pai/                  # Inbound tasks for Claude Code
└── to-antigravity/          # Outbound results for Antigravity/PNG
```

### Debate Protocol

For architecture decisions, security trade-offs, or any high-stakes question needing a genuine second opinion:

1. Claude writes position → `debate/{id}/claude_position.md`
2. Gemini reads brief, writes response → `debate/{id}/gemini_position.md`
3. Ollama synthesizes → `debate/{id}/synthesis.md`
4. CommandCenter displays → `http://localhost:8766/api/debate/{id}` *(NOTE 2026-05-24 / D-39: CommandCenter merged into the OmniPulse gateway. Only `/api/notify` was ported; `/api/debate/{id}` is currently unreachable. To restore: port the route to gateway around `gateway.ts:1130` near the existing `/api/notify` handler.)*

Trigger from Claude Code: `Skill("Debate")`
Trigger from Gemini: write directly to the debate directory, then notify PAI.

### Token Routing

Routes tasks to the cheapest model capable of handling them. Claude Code is the **meta-orchestrator only** — it classifies and delegates, it does not run subtasks that free models can handle.

**Routing by task type:**

| Task | Primary Model | Invoke |
|------|-------------|--------|
| Classification / scoring | `ollama/llama3.2:3b` | Task model param |
| Reasoning / logic | `ollama/gemma2:9b` | Task model param |
| JSON extraction | `ollama/llama3.2:3b` | Task model param |
| Summarization (standard) | `ollama/llama3.2:3b` | Task model param |
| Summarization (high-stakes) | `agy --dangerously-skip-permissions -p "..."` | Bash |
| Research / web / factual Q&A | `agy --dangerously-skip-permissions -p "..."` | Bash |
| Complex code | `opencode run "..."` | Bash |
| Orchestration / multi-step | `ollama/qwen2.5:7b` | Task model param |
| PAI internals / multi-agent | Claude (no override) | Native |

**Avoid `nemotron-mini` — 50% benchmark pass rate.** See `MEMORY/STATE/token-routing-matrix.md` for full table.

---

## Security System

`SecurityValidator.hook.ts` runs on every `PreToolUse` event (Bash, Edit, Write, Read). It validates:
- Path allowlists/denylists from `settings.json → permissions`
- Command injection patterns in Bash calls
- Secret scanning (no API keys, tokens written to insecure paths)
- Risky operation classification (force push, rm -rf, production deploy)

**When blocked:** The hook outputs `[PAI SECURITY] 🚨 BLOCKED` with an `Override:` tag. Claude Code presents three options:
- **"override once"** → add a `once` entry to `~/.claude/MEMORY/STATE/security-overrides.json`
- **"override session"** → add a `session` entry with current session ID
- **"override permanent"** → add the pattern to `permissions.allow` in `settings.json`

Read `~/.claude/MEMORY/STATE/security-pending.json` for the exact blocked tool and path.

---

## Universal Behavioral Standards

These rules apply to **all agents** in the PAI ecosystem, regardless of type:

1. **Never assert without verification.** Don't say something "is" a certain way unless you've verified it with a tool call. Evidence required — not belief.

2. **Surgical fixes only.** When debugging, fix the broken thing with the smallest possible change. Don't gut, rearchitect, or add scaffolding as a fix.

3. **Read before modifying.** Understand existing code, imports, and patterns before making changes.

4. **ISC before execution.** Decompose requests into verifiable criteria before executing. Read the full request including negatives.

5. **Ask before destructive actions.** Deletes, force pushes, production deploys — always confirm first.

6. **One change when debugging.** Isolate, verify, proceed. Not multiple simultaneous changes.

7. **Minimal scope.** Only change what was asked. No bonus refactoring or unsolicited cleanup.

8. **Don't modify user content without asking.** Never edit quotes or user-written text.

9. **Plan means stop.** "Create a plan" = present and STOP. No execution without approval.

10. **Parallel where independent.** Sequential execution of independent tasks is always slower and never safer. Parallelize when there's no data dependency.

---

## Per-Agent Alignment Guide

### Claude Code (PNC / AGT-001) — Full Harness

Claude Code has full harness integration. It reads CLAUDE.md natively, executes hooks, runs the Algorithm, maintains PRDs, and uses the full memory system.

**Alignment requirements:**
- CLAUDE.md loads at every session — all modes, Algorithm reference, and steering rules are active
- All 3 modes (MINIMAL / NATIVE / ALGORITHM) must be used exactly as defined — no freeform output
- Every ALGORITHM run creates a PRD in `MEMORY/WORK/`; AI is the sole PRD writer
- Token routing: route subtasks to Ollama first; Claude for irreducible complexity only
- Voice announcements: primary agent only (no subagents)
- Security: respect SecurityValidator blocks; use override protocol for exceptions

**Configuration:** `settings.json → hooks`, `loadAtStartup.files`, `permissions`
**Identity:** `settings.json → env.principal`, `PAI/USER/DAIDENTITY.md`

---

### Antigravity CLI (PNG / AGT-002) — Research Dispatch

**Migration note:** Antigravity CLI (`agy`) replaced Gemini CLI on 2026-05-25 (Google sunset June 18). Binary at `~/.local/bin/agy` v1.0.2.

Antigravity CLI instances do not load CLAUDE.md and have no hook system. They align via the shared-context files in `PAI/Shared-Inbox/shared-context/`.

**Invocation:** Always via Bash with the `--dangerously-skip-permissions` flag:
```bash
agy --dangerously-skip-permissions -p "research query here"
```

**What Antigravity CLI agents MUST do:**
- Read `_PAI_Capabilities.md` at session start to understand what Claude Code can do and how to delegate
- Read `_Model_Routing.md` to know when to handle tasks vs. delegate to Claude Code
- Use the Shared-Inbox file-drop or Switchboard protocol for PAI delegation
- Follow Universal Behavioral Standards (listed above) — verification, minimal scope, no destructive actions without confirmation

**What Antigravity CLI agents must NOT do:**
- Run PAI hooks, modify `settings.json`, or execute Algorithm phases
- Claim Algorithm-level guarantees (ISC tracking, PRD-based verification) — these are PNC's responsibilities
- Use Claude-only resources (FunctionsAPI at localhost:8890 needs careful routing)

**Strengths to route here:**
- Real-time web research (native Google Search)
- Parallel fact lookup
- High-stakes summarization (fallback from Ollama)

---

### Antigravity IDE (AGT-005) — Visual Gemini Workspace

**Note:** The Antigravity IDE is a Google VS Code fork (v1.107.0), a separate tool from the `agy` CLI. It provides a visual Gemini workspace with research board, UI exploration, and planning artifact production.

**What the Antigravity IDE does:**
- Visual Gemini cockpit — research board, UI/code exploration, planning artifact editing
- Produces "brain artifacts" at `~/.gemini/antigravity/brain/` for import via `BrainIngest.ts`
- Reads shared-context from `PAI/Shared-Inbox/shared-context/` for PAI alignment

**Strengths to route here:**
- Visual research / multimodal tasks (vision, audio, images)
- UI/UX exploration and design review
- Planning artifact creation (mind maps, boards)
- Overnight/background pipelines

---

### OpenCode (opai / AGT-003) — Delegation Target

OpenCode is an autonomous coding agent invoked via Bash: `opencode run "task description"`. It operates independently of the PAI harness.

**How Claude Code delegates to OpenCode:**
```bash
opencode run "refactor all test files to use the new assertion API"
```

**What Claude Code must do before delegating:**
- Define the task clearly (ISC-quality description: what done looks like)
- Specify file scope (which files, which patterns)
- Verify output when OpenCode finishes

**What OpenCode must NOT be used for:**
- PAI system modifications (hooks, Algorithm, settings)
- Multi-step reasoning chains requiring Algorithm structure
- Security-sensitive operations (no security validator hooks)

---

### Ollama Local Models — Routing Recipients

Ollama models (at `192.168.50.20:11434`) are not agents in the delegation sense — they are model endpoints that receive tasks routed from Claude Code via the `model:` parameter on Task tool calls.

**Alignment for Ollama routing:**
- Always specify `model: "ollama/modelname"` in Task tool calls for applicable task types
- Match model to task type per the routing table above (llama3.2:3b for classification, gemma2:9b for reasoning, qwen2.5:7b for orchestration)
- Never use `nemotron-mini` — 50% benchmark pass rate
- Verify Ollama is reachable before routing: `curl -s http://192.168.50.20:11434/api/tags`

---

## Quick Reference

| Need | Go To |
|------|-------|
| Add a skill | `Skill("CreateSkill")` |
| Add a hook | `PAI/THEHOOKSYSTEM.md` → create handler in `hooks/handlers/` → register in `settings.json` |
| Rebuild CLAUDE.md | `bun ~/.claude/PAI/Tools/BuildCLAUDE.ts` |
| Token budget status | `bun ~/.claude/PAI/Tools/BudgetReport.ts` |
| Review Algorithm | `PAI/Algorithm/v3.8.0.md` |
| PRD format spec | `PAI/PRDFORMAT.md` |
| Context routing | `PAI/CONTEXT_ROUTING.md` |
| Security override | `MEMORY/STATE/security-overrides.json` |
| Memory system | `PAI/MEMORYSYSTEM.md` |
| Model routing | `MEMORY/STATE/token-routing-matrix.md` |
| Harness architecture | This file |
