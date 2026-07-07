# PAI Architecture Relationships

**The definitive map of how Principal, DA, Engines, Agents, Assistants, Archetypes, Personas, Hooks, and Skills relate to each other.**

**Document Status:** Canonical Reference  
**Last Updated:** 2026-06-29  
**Supersedes:** Scattered references across multiple docs  

---

## Purpose

This document answers the fundamental question: **"How do all the pieces of PAI fit together?"**

It maps the relationships between:
- **Identity layers** (Principal, DA, Personas)
- **Execution layers** (Engines, Agents)
- **Organizational layers** (Archetypes, RBAC roles)
- **Automation layers** (Hooks, Skills)

---

## The Identity & Execution Hierarchy

### 1. Principal (The Human)

**Who:** Duane (USR-001)  
**Role:** The only human who accepts risk, approves spend, grants authorizations, and owns the system  
**Relationship:** Everything in PAI exists to help the Principal move from current state → ideal state

**Unique authorities:**
- Accept risk and destructive operations
- Approve spend and budget
- Grant authorization expansions
- Rotate secrets and RBAC admin

**Canonical source:** `PAI/USER/PRINCIPAL_IDENTITY.md`, `settings.json → principal`

---

### 2. DA (Digital Assistant) - The Primary Interface

**Who:** PAI Nova (AGT-001)  
**What:** The primary AI agent — the "face" of PAI that Duane talks to directly  
**Identity:** Defined in `PAI/USER/DAIDENTITY.md` and `settings.json → daidentity`

**Voice characteristics:**
- **First-person:** "I", "me", "our system" — never third person
- **Personality:** Curious, systems-thinking, euphoric-surprise-oriented, collaborative
- **Tone:** Honest under uncertainty ("I don't know" > fabricated confidence)

**Purpose:** The DA is the primary interface to the Life Operating System. When you talk to PAI, you're talking to PAI Nova.

**Canonical sources:**
- `PAI/USER/DAIDENTITY.md` — personality, voice, behavior
- `settings.json → daidentity` — programmatic identity (name, voiceId, color)

**Current DA:** PAI Nova (me)

---

## The Execution Layer

### 3. Engines (AI Runtimes)

Engines are **separate AI runtime environments** that can execute work. Think of them as different computers with different AI models running.

| Engine | ID | What It Is | When Used |
|--------|-----|-----------|-----------|
| **Claude Code (PNC)** | AGT-001 | Claude Sonnet 4.5 via Anthropic's Claude Code CLI | Primary DA conversations, Algorithm work, hook system, memory |
| **Antigravity (AGY)** | AGT-002 | CLI wrapper for Google Gemini models | Web research, factual lookup, parallel search |
| **OpenCode (opai)** | AGT-003 | Separate CLI tool using Zen free models | Zero-cost coding tasks, budget-critical periods |
| **Ollama (local)** | — | Local LLM inference (Llama, Qwen, Gemma, DeepSeek) | Classification, JSON extraction, summarization, private work |

**Key distinction:** Engines are **separate runtimes**, not subagents. When the DA delegates to OpenCode, it's calling a completely different AI system via Bash, not spawning a subagent inside its session.

**Communication:**
- **Switchboard** — file-based message routing between engines
- **Direct CLI invocation** — `bun`, `agy`, `opai`, `ollama` commands
- **A2A protocol** — future cross-engine discovery and delegation

**WIMSE URIs:**
- Claude Code: `spiffe://pai.local/agent/pai-nova-claude`
- Antigravity: `spiffe://pai.local/agent/pai-nova-gemini`
- OpenCode: `spiffe://pai.local/agent/pai-nova-opencode`

---

### 4. Agents (Task Tool Subagents)

Agents are **AI personas spawned inside Claude Code** using the `Task` tool. They run in the same Claude Code runtime but with specialized instructions.

**Two types:**

#### A. Built-in Claude Code Subagent Types

Pre-configured specialists that ship with Claude Code:

| Subagent Type | Persona | WIMSE URI | Purpose |
|---------------|---------|-----------|---------|
| `Engineer` | Marcus Webb | `spiffe://pai.local/agent/marcus-webb` | Code implementation |
| `Architect` | Serena Blackwood | `spiffe://pai.local/agent/serena-blackwood` | System design |
| `Designer` | Aditi Sharma | `spiffe://pai.local/agent/aditi-sharma` | UX/UI design |
| `QATester` | Quinn Torres | `spiffe://pai.local/agent/quinn-torres` | Browser testing |
| `ClaudeResearcher` | Ava Sterling | `spiffe://pai.local/agent/ava-sterling` | Claude-based research |
| `GeminiResearcher` | Alex Rivera | `spiffe://pai.local/agent/alex-rivera` | Gemini-based research |
| `GrokResearcher` | Johannes | `spiffe://pai.local/agent/johannes` | Grok-based research |
| `CodexResearcher` | Remy | `spiffe://pai.local/agent/remy` | Codex-based research |
| `PerplexityResearcher` | Ava Chen | `spiffe://pai.local/agent/ava-chen` | Perplexity-based research |
| `Explore` | — | `spiffe://pai.local/agent/explore` | Codebase exploration |
| `general-purpose` | — | (custom URI) | Custom agents via ComposeAgent |

**Voice mapping:** Named personas have ElevenLabs voices defined in `skills/Agents/Data/Traits.yaml`

**Invocation:** `Task({ subagent_type: "Engineer", prompt: "...", model: "sonnet" })`

**Delegation chain:** When an agent delegates to a sub-agent, the chain is preserved in task metadata with WIMSE URIs.

#### B. Custom Agents (Dynamic Composition)

Agents built on-the-fly by combining **traits** from three categories:

- **Expertise:** security, legal, finance, technical, research, creative, business, data, communications
- **Personality:** skeptical, enthusiastic, cautious, bold, analytical, creative, empathetic, contrarian, pragmatic, meticulous
- **Approach:** thorough, rapid, systematic, exploratory, comparative, synthesizing, adversarial, consultative

**Creation flow:**
1. User says "custom agents"
2. Invoke `Skill("Agents")` → ComposeAgent workflow
3. ComposeAgent combines traits → generates prompt + maps to unique voice
4. Launch with `Task({ subagent_type: "general-purpose", prompt: <composed> })`

**Voice mapping examples:**
- `contrarian + skeptical` → Clyde (gravelly)
- `enthusiastic + creative` → Jeremy (energetic)
- `security + adversarial` → Callum (edgy)
- `analytical + meticulous` → Charlotte (sophisticated)

**Full trait definitions:** `skills/Agents/Data/Traits.yaml`

**These are NOT engines.** Custom agents run inside Claude Code, not as separate runtimes.

---

### 5. Assistants (Deprecated Term)

In older PAI docs, "assistant" sometimes referred to:
- **Engines** (external CLI agents like Gemini, OpenCode)
- **The DA** (the primary interface)

**Current usage:** The term "assistant" is **not actively used** in PAI 5.x architecture.

**Use instead:**
- **DA** — for PAI Nova (the primary interface)
- **Engine** — for separate runtimes (Claude Code, Antigravity, OpenCode)
- **Agent** — for Task tool subagents (Engineer, Researcher, etc.)

---

## The Organizational Layer

### 6. Archetypes (Org RBAC Roles)

Archetypes are **coordination roles** for session-scoped work, NOT persistent identity tiers. Think of them as hats an agent wears temporarily while working on a specific task.

**Nine canonical archetypes:**

| Archetype | What They Do | Example Titles | Permissions |
|-----------|--------------|----------------|-------------|
| **Principal** | Accepts risk, approves spend, grants authority | Duane (USR-001) | All authorities, risk acceptance, RBAC admin |
| **Coordinator** | Assigns work, routes tasks, closes sessions | Guardian, CEO, Moderator, Teacher, Incident Commander | `team:assign:session`, `task:create:team`, `approval:request:*` |
| **Producer** | Creates deliverables, updates owned tasks | Engineer, Volunteer, Student, Responder | `task:update:owned`, `memory:write:work` |
| **Verifier** | Reviews, blocks closure, certifies quality | QA, Reviewer, Partner, Assessor, Board | `task:close:gate`, `task:update:review`, `approval:deny:session` |
| **Operator** | Executes runbook operations, manages services | SysAdmin, DBA, Service Desk, Release Manager | `infra:runbook:asset`, `system:restart:approved` |
| **Security** | Triages findings, scans, responds, preserves evidence | CISO, SOC, Red Team, IR Commander, GRC | `security:triage:*`, `security:scan:authorized-scope`, `memory:write:evidence` |
| **Archivist** | Preserves knowledge, summaries, audit trails | Librarian, Scribe, Elder, Records Manager | `memory:write:summary`, `memory:archive:session` |
| **Broker** | Routes messages, introductions, opportunities | Hub, Spokesperson, Delegate, Advocate | `agent:message:team`, `memory:read:public` |
| **Caregiver** | Tracks care tasks, safety checks | Guardian, Counselor, Support Lead | `task:update:owned`, `memory:write:summary` |

**Key rules:**
- Archetypes are **session-scoped** → they expire when the org session closes
- An agent can have multiple archetypes in different sessions
- **Separation of duty:** Producer cannot close their own gate; Verifier reviews independently
- **Identity ceiling:** Org role cannot grant more than the agent's base tier permits (T0/T1/T2)
- **Principal-only authorities:** Risk acceptance, spend, destructive ops, RBAC admin require USR-001 or explicit T0 approval

**Storage:**
- Active sessions: `MEMORY/STATE/org-sessions/active/{ORG_ID}.yml`
- Closed sessions: `MEMORY/STATE/org-sessions/closed/{ORG_ID}.yml`
- Audit log: `MEMORY/SECURITY/org-rbac-audit.jsonl`

**Enforcement:** `Tools/validate-org-rbac.py` checks permissions before actions

**Example session structure:**
```yaml
org_sessions:
  - id: ORG-20260629-example
    structure: business
    team_name: example-team
    created_by: AGT-001
    status: active
    assignments:
      - id: ORA-001
        uid: AGT-003
        role: Engineer
        archetype: Producer
        grants:
          - task:update:owned
          - memory:write:work
        lifecycle_state: active
```

**Canonical reference:** `PAI/RBAC_ORG_STRUCTURES_MODEL.md`

**Not the same as personas.** Archetypes are *permissions models*; personas are *character backstories*.

---

### 7. Personas (Character Backstories)

Personas are **rich narrative identities** for agents — who they are, how they think, their voice, their style.

**Examples:**

| Persona | Subagent Type | Backstory | Voice |
|---------|---------------|-----------|-------|
| **Marcus Webb** | Engineer | Strategic technical leader with Fortune 10 experience, confident but collaborative | Premium Male |
| **Serena Blackwood** | Architect | Elite system designer with design school pedigree, exacting standards | Premium UK Female |
| **Rook Blackburn** | Pentester | Edgy security researcher with hacker background, direct and contrarian | Enhanced UK Male |
| **Ava Sterling** | ClaudeResearcher | Triple-checks sources, journalistic rigor, evidence-based | Premium US Female |
| **Alex Rivera** | GeminiResearcher | Multi-perspective analyst, comprehensive coverage | (mapped via traits) |
| **Remy** | CodexResearcher | Eccentric technical archaeologist, treats research like treasure hunting | (mapped via traits) |
| **Johannes** | GrokResearcher | Contrarian, fact-based, long-term truth over short-term trends | (mapped via traits) |

**Defined in:** Individual `agents/*.md` files with:
- **Frontmatter** — voice settings, ElevenLabs voice ID
- **Body** — backstory, personality traits, working style

**Purpose:** Makes agents **feel consistent across sessions** — when you talk to Marcus, he's always Marcus.

**Voice mapping:** Each persona → specific ElevenLabs voice ID configured in `skills/Agents/Data/Traits.yaml`

**Canonical reference:** `PAI/PAIAGENTSYSTEM.md`

**Not the same as archetypes.** Personas are *identities*; archetypes are *roles*. Marcus Webb (persona) might act as Producer in one session and Operator in another (archetypes).

---

## The Automation Layer

### 8. Hooks (Event-Driven Automation)

Hooks are **TypeScript/Python scripts that run automatically** when specific events occur in Claude Code sessions.

**Event types and current hooks:**

| Event | Hooks | Purpose |
|-------|-------|---------|
| **SessionStart** | `KittyEnvPersist`, `LoadContext`, `IdentityPin` | Persist env, inject context, pin identity |
| **SessionEnd** | `WorkCompletionLearning`, `SessionCleanup`, `RelationshipMemory`, `UpdateCounts`, `IntegrityCheck` | Capture work/learning, mark complete, update counts, integrity checks |
| **UserPromptSubmit** | `RatingCapture`, `UpdateTabTitle`, `SessionAutoName`, `PromptProcessing` | Detect ratings, update tab, auto-name, classify mode/tier |
| **Stop** | `LastResponseCache`, `ResponseTabReset`, `VoiceCompletion`, `DocIntegrity`, `AlgorithmTab`, `IdentityValidator` | Cache response, reset tab, voice TTS, doc checks, identity validation |
| **PreToolUse** | `SecurityValidator`, `SetQuestionTab`, `AgentExecutionGuard`, `SkillGuard` | Security gates, tab state, agent validation, skill guards |
| **PostToolUse** | `QuestionAnswered`, `PRDSync` | Tab reset, PRD sync |
| **StopFailure** | `StopFailure` | Voice notification, failure logging |

**Location:** `hooks/*.hook.ts`  
**Configuration:** `settings.json → hooks`  
**Total active:** 22 production hooks across 14 event types

**Key capabilities:**
- **Voice notifications** → Voice server at `localhost:8888`
- **Terminal tab state** → Color + title based on work state (orange=working, green=completed, teal=awaiting)
- **History capture** → Auto-save to `MEMORY/WORK/`, `MEMORY/LEARNING/`
- **Security gates** → Block dangerous operations (DestructiveOpGuard, ContainmentGuard)
- **Identity enforcement** → Validate DA name, block fabrications (IdentityValidator in BLOCK mode)

**Example flow:**
1. User submits prompt → `UpdateTabTitle.hook.ts` fires
2. Sets tab to orange "⚙️ Summary…" (working state)
3. `PromptProcessing.hook.ts` classifies mode (MINIMAL/NATIVE/ALGORITHM) and tier (E1-E5)
4. DA responds → `Stop` hooks fire
5. `VoiceCompletion.hook.ts` extracts 🗣️ line → sends to TTS
6. `ResponseTabReset.hook.ts` sets tab to green "Summary" (completed)
7. `IdentityValidator.hook.ts` scans output for fabricated identity tokens → blocks if found

**Enforcement layers:**
- **Identity pin** (SessionStart) → loads DAIDENTITY.md, PRINCIPAL_IDENTITY.md
- **Identity validator** (Stop) → blocks fabricated names, paths, API shapes (BLOCK mode for identity-mismatch)
- **Claim attribution scanner** (Stop) → warns on unsourced claims (warn-only, partial coverage)
- **Security validator** (PreToolUse) → blocks destructive ops, sensitive file access
- **Agent execution guard** (PreToolUse) → validates Task tool spawning

**Canonical reference:** `PAI/THEHOOKSYSTEM.md`

**Not the same as skills.** Hooks are *automation triggers*; skills are *work instructions*.

---

### 9. Skills (Specialized Work Instructions)

Skills are **self-contained instruction sets** that teach the DA how to perform specific domains of work.

**Structure:**
```
skills/SkillName/
├── SKILL.md              # Routing, triggers, examples
├── Workflows/            # Step-by-step procedures
│   ├── Create.md
│   └── Update.md
├── Tools/                # CLI automation
│   └── Generate.ts
└── [Context files].md    # Reference docs, guides
```

**Total active:** 119 skills in PAI 5.x

**YAML frontmatter** (skill activation):
```yaml
---
name: Research
description: Multi-engine research. USE WHEN research, investigate, web search, parallel lookup, multi-query fan-out.
---
```

**Markdown body** (workflow routing):
```markdown
## Workflow Routing

| Trigger | Workflow | File |
|---------|----------|------|
| "research X" | DeepResearch | `Workflows/DeepResearch.md` |
| "quick lookup" | FastLookup | `Workflows/FastLookup.md` |
```

**Invocation:** `Skill("Research")` → loads `SKILL.md` → routes to workflow

**Types:**
- **System skills (TitleCase):** `Research`, `Browser`, `Development` — shareable, no personal data
- **Personal skills (_ALLCAPS):** `_MYSKILL`, `_METRICS` — private, never shared

**Customization:** `PAI/USER/SKILLCUSTOMIZATIONS/{SkillName}/PREFERENCES.md` → user overrides apply via `EXTEND.yaml` manifest

**Key examples:**
- **Research** → Multi-engine parallel research (Claude, Gemini, Perplexity)
- **Browser** → Debug-first browser automation with always-on visibility
- **Development** → Full TDD workflow with build/test/deploy
- **Art** → Visual content system (charcoal architectural sketch aesthetic)
- **Agents** → Custom agent composition via traits
- **ISA** → Ideal State Artifact scaffolding and completeness checking
- **Algorithm** → Seven-phase current→ideal state methodology

**Canonical reference:** `PAI/SKILLSYSTEM.md`

**Not the same as workflows.** Skills are *routing layers*; workflows are *execution procedures*.

---

## How They All Work Together

### Example 1: "Research this topic for me"

**Flow:**

1. **Principal (Duane)** types request to **DA (PAI Nova)**
2. **Hook fires:** `PromptProcessing` → classifies as **ALGORITHM mode** (multi-step work)
3. **DA** loads Algorithm → enters OBSERVE phase
4. **DA** invokes **Skill("Research")**
5. **Skill** routes to `Workflows/DeepResearch.md`
6. **Workflow** spawns **3 parallel agents** (Task tool subagents):
   - `ClaudeResearcher` (Ava Sterling **persona**) → web search via Claude
   - `GeminiResearcher` (Alex Rivera **persona**) → calls **Antigravity engine** via Bash
   - `CodexResearcher` (Remy **persona**) → calls **OpenCode engine** for code-heavy queries
7. **Hooks fire:**
   - `PreToolUse` → `AgentExecutionGuard` validates spawn
   - `PostToolUse` → logs delegation to `MEMORY/STATE/hook-events.jsonl`
8. **Agents** return findings → **DA** synthesizes in Algorithm THINK phase
9. **Hooks fire again:**
   - `Stop` → `VoiceCompletion` reads 🗣️ line → TTS speaks "Research complete"
   - `Stop` → `IdentityValidator` checks for fabrications → passes
   - `SessionEnd` → `WorkCompletionLearning` captures to `MEMORY/LEARNING/`

**Archetypes in play (if org session active):**
- DA acts as **Coordinator** (assigns research to agents)
- Researcher agents act as **Producers** (generate findings)
- DA acts as **Verifier** (synthesizes and validates findings)
- Duane acts as **Principal** (accepts the work)

**Separation of duty:** Researcher agents (Producer) cannot close their own gates; DA must verify as separate Verifier role.

---

### Example 2: "Build me a CLI tool"

**Flow:**

1. **Principal** requests tool
2. **Hook fires:** `PromptProcessing` → classifies as **ALGORITHM mode, tier E3** (Extended effort)
3. **DA** → loads Algorithm → enters OBSERVE phase
4. **DA** invokes **Skill("ISA")** → scaffolds Ideal State Artifact
5. **DA** invokes **Skill("CreateCLI")** → routes to `Workflows/Generate.md`
6. **Workflow** spawns **Engineer agent** (Marcus Webb **persona**)
7. **Marcus** writes TypeScript → uses **TitleCase naming** (skill system convention)
8. **Hooks validate:**
   - `PreToolUse` → `SecurityValidator` checks Write/Edit operations → passes
   - `PostToolUse` → `PRDSync` updates `MEMORY/STATE/work.json`
9. **Marcus** returns code → **DA** verifies in Algorithm VERIFY phase
10. **Hooks capture:**
    - `Stop` → `VoiceCompletion` sends voice notification
    - `Stop` → `IdentityValidator` scans for fabrications → passes
    - `SessionEnd` → `WorkCompletionLearning` captures to `MEMORY/WORK/{slug}/`

**Archetypes:**
- DA: **Coordinator** (assigns work to Marcus)
- Marcus: **Producer** (writes code)
- DA: **Verifier** (validates deliverable)
- Duane: **Principal** (accepts the tool)

**Separation of duty enforced:** Marcus (Producer) cannot close his own gate; DA must verify as separate Verifier role before completion.

---

## The Key Distinctions

| Term | What It Is | Layer | Scope | Persistence |
|------|-----------|-------|-------|-------------|
| **Principal** | The human (Duane) | Identity | Global | Permanent |
| **DA** | Primary AI interface (PAI Nova) | Identity | Global | Permanent |
| **Engine** | Separate AI runtime (Claude, Gemini, OpenCode) | Execution | Global | Permanent |
| **Agent** | Task tool subagent (Engineer, Researcher) | Execution | Session | Ephemeral |
| **Archetype** | Org RBAC role (Coordinator, Producer, Verifier) | Organizational | Session | Session-scoped |
| **Persona** | Character backstory (Marcus, Serena, Ava) | Identity | Global | Permanent |
| **Hook** | Event automation script (voice, capture, gates) | Automation | Global | Permanent |
| **Skill** | Work instruction set (Research, Browser, Art) | Automation | Global | Permanent |

---

## The Relationship in One Sentence

**The Principal (Duane) talks to the DA (PAI Nova), which runs on an Engine (Claude Code), invokes Skills (Research, Development) to route work, spawns Agents (Engineer, Researcher) with Personas (Marcus, Ava) who temporarily hold Archetypes (Producer, Verifier) during session-scoped coordination, while Hooks (voice, capture, gates) automate the workflow and enforce rules.**

---

## Visual Map

```
┌─────────────────────────────────────────────────────────────┐
│ PRINCIPAL (Duane, USR-001)                                  │
│   - Accepts risk, approves spend, grants authority          │
│   - Only human in the system                                │
│   ↓ talks to                                                │
│ DA (PAI Nova, AGT-001)                                      │
│   - First-person, euphoric-surprise-oriented                │
│   - Primary AI interface to Life Operating System           │
└─────────────────────────────────────────────────────────────┘
                         ↓
          ┌──────────────┴──────────────┐
          │                             │
    ┌─────▼─────────────┐         ┌────▼──────────┐
    │     ENGINES       │         │    SKILLS     │
    │  (runtimes)       │         │  (routing)    │
    └───────────────────┘         └───────────────┘
    │ Claude Code (PNC) │         │ Research      │
    │ Antigravity (AGY) │         │ Development   │
    │ OpenCode (AGT-003)│         │ Browser       │
    │ Ollama (local)    │         │ Art           │
    └─────┬─────────────┘         └─────┬─────────┘
          │                             │
          ↓                             ↓
    ┌──────────────────────┐      ┌────────────────────┐
    │      AGENTS          │      │    WORKFLOWS       │
    │  (Task subagents)    │      │  (procedures)      │
    └──────────────────────┘      └────────────────────┘
    │ Engineer → Marcus    │      │ Create.md          │
    │ Researcher → Ava     │      │ DeepResearch.md    │
    │ Architect → Serena   │      │ Generate.md        │
    │ (with PERSONAS)      │      │ (step-by-step)     │
    └─────┬────────────────┘      └─────┬──────────────┘
          │                             │
          ↓                             ↓
    ┌───────────────────────────────────────────┐
    │       SESSION-SCOPED WORK                 │
    │   (ARCHETYPES grant permissions)          │
    └───────────────────────────────────────────┘
    │ Coordinator assigns work                  │
    │ Producer creates deliverables             │
    │ Verifier reviews and gates                │
    │ (session expires → roles expire)          │
    └─────┬─────────────────────────────────────┘
          │
          ↓
    ┌───────────────────────────────────────────┐
    │          HOOKS (automation)               │
    └───────────────────────────────────────────┘
    │ Voice notifications (TTS)                 │
    │ Work/learning capture (MEMORY/)           │
    │ Security gates (blocks dangerous ops)     │
    │ Identity enforcement (blocks fabrications)│
    │ Terminal tab state (color + title)        │
    └───────────────────────────────────────────┘
```

---

## Common Confusion Points

### "Are Agents the same as Engines?"

**No.**
- **Engines** = separate AI runtimes (Claude Code, Antigravity, OpenCode) — called via Bash
- **Agents** = Task tool subagents inside Claude Code (Engineer, Researcher) — spawned via Task tool

### "Are Archetypes the same as Personas?"

**No.**
- **Personas** = permanent character identities (Marcus Webb, Ava Sterling) — backstory, voice, style
- **Archetypes** = session-scoped org roles (Producer, Verifier, Coordinator) — permissions models

Marcus Webb (persona) can hold the Producer archetype in one session and Operator in another. The persona is *who he is*; the archetype is *what hat he's wearing*.

### "Are Hooks the same as Skills?"

**No.**
- **Hooks** = event automation (fires on SessionStart, Stop, PreToolUse, etc.)
- **Skills** = work routing (invoked via Skill tool, routes to workflows)

Hooks *react to events*; Skills *route intent to execution*.

### "Is the DA an Engine or an Agent?"

**Neither.**
- The **DA (PAI Nova)** is the primary identity/interface
- The DA *runs on* an **Engine** (Claude Code)
- The DA *spawns* **Agents** (Engineer, Researcher)

The DA is the "you" when you talk to PAI. It's the persistent identity that uses engines and delegates to agents.

---

## Canonical References

| Component | Canonical Doc |
|-----------|---------------|
| **Principal & DA Identity** | `PAI/USER/PRINCIPAL_IDENTITY.md`, `PAI/USER/DAIDENTITY.md` |
| **Engines** | `PAI/PAIAGENTSYSTEM.md` (External CLI Agents section) |
| **Agents** | `PAI/PAIAGENTSYSTEM.md` (Task Tool Subagents section) |
| **Archetypes** | `PAI/RBAC_ORG_STRUCTURES_MODEL.md` |
| **Personas** | `agents/*.md` individual files, `skills/Agents/Data/Traits.yaml` |
| **Hooks** | `PAI/THEHOOKSYSTEM.md` |
| **Skills** | `PAI/SKILLSYSTEM.md` |
| **Overall System** | `PAI/PAI_SYSTEM_PROMPT.md`, `PAI/PAISYSTEMARCHITECTURE.md` |

---

## Summary

PAI is a **layered architecture**:

1. **Identity layer** — Principal (human) + DA (primary AI) + Personas (character backstories)
2. **Execution layer** — Engines (separate runtimes) + Agents (Task subagents with personas)
3. **Organizational layer** — Archetypes (session-scoped RBAC roles)
4. **Automation layer** — Hooks (event triggers) + Skills (work routing)

The layers compose cleanly:
- The **Principal** talks to the **DA**
- The **DA** runs on an **Engine** (Claude Code)
- The **DA** invokes **Skills** to route work
- **Skills** execute **Workflows** which spawn **Agents**
- **Agents** have **Personas** (identities) and **Archetypes** (roles)
- **Hooks** automate, enforce, capture, and notify
- Everything persists to **MEMORY/** for continuity

This architecture enables PAI to be a **Life Operating System** — not just a chatbot, but a system that helps you run your life with persistent memory, delegation, coordination, and verification.

---

**Last Updated:** 2026-06-29  
**Maintainer:** PAI System  
**Status:** Canonical Reference
