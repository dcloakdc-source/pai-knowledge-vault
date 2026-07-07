# PAI Glossary & Index

**Purpose:** Single lookup surface for every acronym, term, codename, and concept used across the PAI infrastructure. Each entry gives the expansion (for acronyms), a one-to-two-sentence definition, aliases, and the canonical source file the definition derives from.

**Created:** 2026-07-02 · **Updated:** 2026-07-02
**Maintenance:** This is a living document. New doctrine terms route here via the LEARN-phase Learning Router (`doctrine`/`rule` types); when you coin or encounter an undefined term, add it with a source path. Definitions were extracted verbatim-first from sources read on the creation date — if an entry and its source ever disagree, the source wins; fix the entry.

**Scope:** Infrastructure terms only. Individual skills are NOT enumerated (127 skill directories exist; the skill registry and `SkillSearch` serve that lookup) — only the skill *system* and a handful of infrastructure-grade skills appear here. Duane's personal definitions live separately in `PAI/USER/DEFINITIONS.md`.

---

## Acronym Quick Reference

| Acronym | Expansion | See entry |
|---------|-----------|-----------|
| AGT | Agent ID prefix (AGT-001…) | [AGT IDs](#a) |
| AGY | Antigravity (Gemini CLI wrapper) | [AGY](#a) |
| DA | Digital Assistant | [DA](#d) |
| E1–E5 | Effort tiers (Standard→Comprehensive) | [Effort Tiers](#e) |
| ICM | Infinite Context Memory | [ICM](#i) |
| ISA | Ideal State Artifact | [ISA](#i) |
| ISC | Ideal State Criteria | [ISC](#i) |
| MOC | Map of Content (knowledge index) | [KnowledgeHarvester](#k) |
| ORA | Org Role Assignment | [ORA](#o) |
| PAI | Personal AI Infrastructure | [PAI](#p) |
| PNC | PAI Nova Claude (Claude Code engine) | [PNC](#p) |
| PNG | PAI Nova Gemini (antigravity/agy engine) | [PNG](#p) |
| PNK | PAI Nova OpenCode (opencode engine) | [PNK](#p) |
| PNO | PAI Nova Ollama (local-model engine) | [PNO](#p) |
| PNX | PAI Nova Codex (codex engine) | [PNX](#p) |
| PRD | Product Requirements Document (ISA's format lineage) | [PRD](#p) |
| RBAC | Role-Based Access Control (two-layered) | [RBAC](#r) |
| RTK | Rust Token Killer | [RTK](#r) |
| USR | User ID prefix (USR-001 = Duane) | [Principal](#p) |

---

## A

- **Activated** — *Core Concept* — Human 3.0 stage 2: "you are acting on the gap." — `PAI/DOCUMENTATION/PAISystemPhilosophy.md`
- **Actualized** — *Core Concept* — Human 3.0 stage 4: "the system and the human operate as one toward the ideal state." — `PAI/DOCUMENTATION/PAISystemPhilosophy.md`
- **Advisor** — *Doctrine* — The Rule 2 reviewer: `bun PAI/Tools/Inference.ts --mode advisor --auto-state` (Opus, 30-min timeout). Auto-state synthesis "closes the biggest RedTeam flaw where the caller could omit problem areas from what the reviewer sees." Mandatory at the E2+ PLAN→BUILD boundary. — `PAI/Algorithm/` (LATEST)
- **Agent (subagent)** — *Component* — "AI personas spawned inside Claude Code using the Task tool. They run in the same Claude Code runtime but with specialized instructions." Distinct from Engines (separate runtimes). Definitions live in `agents/*.md` (Architect, Engineer, Explore, Intern, RealityChecker, researchers, etc.). — `PAI/ARCHITECTURE_RELATIONSHIPS.md`
- **Agent Systems (three)** — *Component* — PAI has three: Task-tool subagent types (internal workflow), Named Agents (persistent identities with ElevenLabs voices), and Custom Agents (composed via ComposeAgent from traits). "Confusing them causes routing failures." — `PAI/PAIAGENTSYSTEM.md`
- **AgentLogger** — *Tool* — "Appends agent activity to the centralized agent-activity.jsonl log. Required by CLAUDE.md for tracking significant tasks and phase shifts"; feeds the dashboard agent view. — `PAI/Tools/AgentLogger.ts`
- **AGT IDs** — *Codename* — Stable agent identifiers: AGT-001 PNC/Claude Code, AGT-002 AGY/Antigravity, AGT-003 opai/OpenCode, AGT-004 Switchboard-Broker, AGT-005 Antigravity IDE, AGT-007 PNO/Ollama, AGT-008 PNX/Codex. — `PAI/ARCHITECTURE_RELATIONSHIPS.md`, `PAI/ASSET_REGISTRY.md`
- **AGY** — *Codename* — Antigravity (AGT-002): "CLI wrapper for Google Gemini models | Web research, factual lookup, parallel search." CLI command `agy`; replaced Gemini CLI 2026-05-25; a shim maps legacy `--yolo` to `--dangerously-skip-permissions`. — `PAI/ARCHITECTURE_RELATIONSHIPS.md`
- **The Algorithm** — *Doctrine* — "The 7-phase execution engine: Observe, Think, Plan, Build, Execute, Verify, Learn. Transitions from CURRENT STATE to IDEAL STATE via verifiable criteria (ISC)." Current version string is read from `PAI/Algorithm/LATEST` (v5.7.11 at writing). Only the primary DA may run ALGORITHM mode. — `PAI/README.md`, `PAI/Algorithm/`
- **ALGORITHM mode** — *Convention* — One of three output modes: "everything else. Including any build/create/make/implement/design/refactor/migrate/integrate request." See Modes. — `PAI/PAI_SYSTEM_PROMPT.md`
- **Aligned** — *Core Concept* — Human 3.0 stage 3: "your daily work and your Telos point the same direction." — `PAI/DOCUMENTATION/PAISystemPhilosophy.md`
- **Anti-criterion** — *Doctrine* — ISC prose-prefix kind: `ISC-N: Anti: <what must NOT happen>`. "≥1 required — a goal with zero failure modes worth naming is under-specified." — `PAI/Algorithm/` (LATEST)
- **Antecedent** — *Doctrine* — ISC prose-prefix kind: a "precondition that reliably produces the target experience (novel juxtaposition, elegance in constraint…)"; ≥1 required when the goal is experiential. — `PAI/Algorithm/` (LATEST)
- **Anvil** — *Codename* — "Kimi K2.6 via Moonshot, 256K context — invoke at E3/E4/E5 when whole-project context materially affects correctness (cross-file refactors, architecture-fitting changes, long-range reasoning)." — `PAI/Algorithm/` (LATEST)
- **aorus** — *Infra* — "Physical machine hosting pai-primary; GPU compute (RTX 3060 6GB + RTX 4060 8GB)." Runs Proxmox; shares its NIC with pai-primary ("NOT separate compute nodes"). — `PAI/ASSET_REGISTRY.md`
- **Archetype** — *Org* — Authority archetypes are the permission primitives of Org RBAC: "the archetype decides the permission shape; the title remains display/context." The nine: Principal, Coordinator, Producer, Verifier, Operator, Security, Archivist, Broker, Caregiver. — `PAI/RBAC_ORG_STRUCTURES_MODEL.md`
- **Archivist** — *Org* — Archetype (aka Librarian, Scribe, Memory Curator): "Preserve knowledge, summaries, decisions, audit trails. Cannot expand access or mutate source facts." — `PAI/RBAC_ORG_STRUCTURES_MODEL.md`
- **ASPIRATIONAL** — *Doctrine* — Enforcement Map status: "only prose; no deterministic enforcement. Surfaced here so it can be promoted to ENFORCED or honestly demoted." — `PAI/ENFORCEMENT_MAP.md`
- **ASSET_REGISTRY.md** — *Convention* — Auto-loaded registry of "hosts, services, ports, paths"; auto-catalogued via SessionEnd hook. First stop for infra facts (paired with `pai-infra.ts resolve`). — `PAI/ASSET_REGISTRY.md`, `PAI/CONTEXT_ROUTING.md`
- **Async Primitive Gate** — *Doctrine* — PLAN gate: "One-shot command → Bash(run_in_background). Event stream → Monitor. AI work → Agent(run_in_background). Never poll in a sleep loop." — `PAI/Algorithm/` (LATEST)
- **Authentik** — *Infra* — SSO proxy on port 9000; "canonical host: m710q." — `PAI/ASSET_REGISTRY.md`
- **Auto-memory** — *Memory* — The harness-level persistent memory at `~/MEMORY/auto/` with the `MEMORY.md` one-line index. Read-side unified via `recall()`; PAI doctrine forbids routing RULES/preferences here (they go to CLAUDE.md/hooks/skills — see Self-Healing Infrastructure). — `PAI/Algorithm/` (LATEST, Learning Router)
- **Aware** — *Core Concept* — Human 3.0 stage 1: "you can see your current state and your ideal state." — `PAI/DOCUMENTATION/PAISystemPhilosophy.md`

## B

- **Backend Falsification Checkpoint** — *Doctrine* — BUILD checkpoint: before building a pipeline that writes to a stateful/external backend, "run a throwaway write+read-back on a disposable key/name FIRST" to confirm the write *method* persists. — `PAI/Algorithm/` (LATEST)
- **Binding Commitment** — *Doctrine* — "Selecting a capability = binding commitment to invoke it via tool… text-only is dishonest and counts as a CRITICAL FAILURE." (aka "no phantom capabilities") — `PAI/Algorithm/` (LATEST)
- **Broker** — *Org* — Archetype (aka Connector, Hub, Delegate, Advocate): "Route messages, introductions, requests, and opportunities. Cannot bind others to obligations." — `PAI/RBAC_ORG_STRUCTURES_MODEL.md`
- **BUILD** — *Doctrine* — Algorithm phase 4/7: preparation work; invoke selected capabilities; write non-obvious decisions to the ISA. Hosts the Root-Cause-at-Ingestion and Backend Falsification checkpoints. — `PAI/Algorithm/` (LATEST)

## C

- **Capability** — *Doctrine* — A skill or agent selected during OBSERVE for the run ("Select what the task genuinely needs within the tier time budget"). Catalogued in `PAI/Algorithm/capabilities.md`. — `PAI/Algorithm/` (LATEST)
- **Capability Routing** — *Convention* — "Design/UI/UX/art/frontend work is capability routing, not a new authority archetype: engage Producer for creation/implementation and Verifier for UX review." — `CLAUDE.md`
- **Caregiver** — *Org* — Archetype (aka Parent, Counselor, Calendar Steward): "Track care tasks, needs, safety checks. No secret/infra/destructive rights by care title." — `PAI/RBAC_ORG_STRUCTURES_MODEL.md`
- **Cato** — *Codename* — The Rule 2a cross-vendor auditor: "runs GPT-5.4 via the codex exec CLI — different vendor… different constitutional training"; mandatory at E4/E5 after the Advisor, read-only sandbox, findings appended to `MEMORY/VERIFICATION/cato-findings.jsonl`. — `PAI/Algorithm/` (LATEST), `PAI/Tools/CrossVendorAudit.ts`
- **CheckpointPerISC** — *Hook* — "Auto-commit on ISC [ ]→[x] transitions… Stages ONLY the ISA file path (never git add -A). Commits with subject ISC-{N} ({slug}): {desc}". Rollback preview via `PAI/Tools/Checkpoint.ts` (prints the `git reset` — never executes it). — `hooks/CheckpointPerISC.hook.ts`
- **ClaimAttributionScan** — *Hook* — "WARN-ONLY telemetry scanner… scans the final assistant message for high-confidence success-claims and cross-checks the session transcript's tool-call log for a corroborating call." Honesty enforcement layer 3 (partial). — `hooks/ClaimAttributionScan.hook.ts`
- **CLAUDE.md** — *Component* — "The master config — generated from CLAUDE.md.template via BuildCLAUDE.ts. It defines execution modes, The Algorithm, and the context routing table." Second in the Context Hierarchy after PAI_SYSTEM_PROMPT.md. — `PAI/README.md`
- **CLI-First Architecture** — *Convention* — "Applies to all new PAI tools, skills, and systems. Philosophy: deterministic code execution > ad-hoc prompting." — `PAI/CLIFIRSTARCHITECTURE.md`
- **ContainmentGuard** — *Hook/Security* — "PreToolUse Edit/Write/MultiEdit gate. Blocks writes that would leak sensitive identity/infra strings into files outside the Z1–Z4 containment zones used by ShadowRelease" — i.e. it blocks hardcoded user-home paths from entering PAI files. — `hooks/ContainmentGuard.hook.ts`
- **Context Hierarchy** — *Doctrine* — "This system prompt defines behavioral non-negotiables: it is the highest authority layer. CLAUDE.md defines operational procedures and format templates." loadAtStartup files come third. — `PAI/PAI_SYSTEM_PROMPT.md`
- **Context Routing** — *Convention* — The on-demand context table: "Load context on-demand by reading the file at the path listed. Only load what the current task requires." — `PAI/CONTEXT_ROUTING.md`
- **Coordinator** — *Org* — Archetype (aka Guardian, Moderator, Incident Commander): "Assign, route, close session decisions, request approvals. Cannot exceed identity tier or bypass gates." — `PAI/RBAC_ORG_STRUCTURES_MODEL.md`
- **Coordination Gate** — *Doctrine* — PLAN gate choosing among the three agent systems: "1. Agent Teams (DEFAULT for parallel work)… 2. Custom Agents (ONLY when Duane says 'custom agents'). 3. Managed Agents (for unattended/overnight work)." — `PAI/Algorithm/` (LATEST)
- **Current State → Ideal State** — *Core Concept* — The universal mechanism: "every task, from shipping code to making art, is a transition from current state to ideal state, pursued through the Algorithm." — `PAI/DOCUMENTATION/PAISystemPhilosophy.md`
- **Custom Agents** — *Component* — "Dynamic agents composed via ComposeAgent from traits — when user says 'custom agents'" (voice trait-mapped). — `PAI/PAIAGENTSYSTEM.md`

## D

- **DA** — *Core Concept* — Digital Assistant: "the primary interface to the Life Operating System. When you talk to PAI, you're talking to PAI Nova." Every PAI user names their own DA; Nova is Duane's. Canonical name source: `settings.json → daidentity.name`. — `PAI/ARCHITECTURE_RELATIONSHIPS.md`, `settings.json`
- **DEFERRED-VERIFY** — *Doctrine* — ISC status marker: "passed in code/intent but live probe is impossible at execution time… Requires a follow-up task ID. Cannot be marked [x] until the deferred probe runs." — `PAI/Algorithm/` (LATEST)
- **Delegation Gate** — *Doctrine* — Before spawning any agent: "Can I do this with Glob + Grep in under 30 seconds? YES → do it directly. NEVER delegate directed lookups… Foreground agent blocking >2 minutes = execution failure." — `PAI/Algorithm/` (LATEST)
- **Deliverable Manifest** — *Doctrine* — PLAN block enumerating "every sub-task the user explicitly asked for… Multi-part requests are the highest-risk failure vector." Mandatory at any tier with 2+ explicit sub-tasks; checked at VERIFY as Deliverable Compliance (any ✗ blocks `phase: complete`). — `PAI/Algorithm/` (LATEST)
- **DestructiveOpGuard** — *Hook/Security* — "PreToolUse:Bash gate. Loads destructive-patterns.json (the LEARNED-PATTERN REGISTRY) and blocks Bash commands that match a learned destructive pattern." Overrides are pattern_slug-keyed (once/session/permanent). Patterns arrive via the promotion pipeline (memo → extractor → validator → dual-factor approval). — `hooks/DestructiveOpGuard.hook.ts`, `PAI/DOCUMENTATION/SECURITY_OVERRIDE_MECHANISM.md`
- **Doc Sync** — *Doctrine* — LEARN step: "if this session modified PAI system files, propagate changes to dependent docs" via the DocumentationUpdate workflow + `DocCheck.ts --changed`. — `PAI/Algorithm/` (LATEST)

## E

- **Effort Tiers (E1–E5)** — *Doctrine* — Standard E1 (<90s, no ISC floor, DEFAULT) · Extended E2 (<3min, ≥16 ISC) · Advanced E3 (<10min, ≥32) · Deep E4 (<30min, ≥128) · Comprehensive E5 (<120min+, ≥256). "The time budget is the hard constraint set by tier; the ISC floor (E2+) is a soft minimum on the count axis only." Explicit override tokens `/e1`–`/e5`. — `PAI/Algorithm/` (LATEST)
- **Effective Permission** — *Org* — "Effective access is the intersection of both layers [Identity RBAC ∩ Org-Scoped RBAC], never their union." — `PAI/RBAC_ORG_STRUCTURES_MODEL.md`
- **EgressClassGuard** — *Hook/Security* — PreToolUse:Bash destination-aware egress watch: warns when a payload's data class (CONFIDENTIAL via secret patterns / PRIVATE via private-tree sources) exceeds the route's ceiling (external = PUBLIC only; internal allows PRIVATE). Warn-only ramp; block mode gated behind `egress-guard.json` + consent after FP soak. Adopted 2026-07-05 from the LifeOS 6.0 delta-mine (ADOPT #1). — `hooks/EgressClassGuard.hook.ts`
- **ENFORCED** — *Doctrine* — Enforcement Map status: "a hook, shim, or other deterministic mechanism blocks or transforms the violating behavior." — `PAI/ENFORCEMENT_MAP.md`
- **Enforcement Map** — *Component* — "Every constitutional rule's coverage is tracked in PAI/ENFORCEMENT_MAP.md — which engines enforce it, which mechanism, current status (ENFORCED / GAP / ASPIRATIONAL)." The source of truth for whether a doctrine layer is live. — `PAI/PAI_SYSTEM_PROMPT.md`
- **Engine** — *Component* — "Separate AI runtime environments that can execute work… Engines are separate runtimes, not subagents." The roster codes: PNC, PNK, PNX, PNG, PNO. — `PAI/ARCHITECTURE_RELATIONSHIPS.md`, `PAI/ENFORCEMENT_MAP.md`
- **EntityGraph** — *Memory* — Entity-proximity graph used to boost `/recall` scoring over the ICM corpus ("FTS5 + entity graph scoring"). — `PAI/Tools/EntityGraph.ts`
- **Euphoric Surprise** — *Core Concept* — The experiential metric: "what you feel when a hard-to-vary explanation meets novelty — an answer that clicks in a way you couldn't have predicted but instantly recognize as true." Predicted and scored 1–10 at THINK (E2+); "if you cannot name an insight, predict ≤6." — `PAI/DOCUMENTATION/PAISystemPhilosophy.md`
- **events.jsonl** — *Memory* — The Unified Event Log: "append-only JSONL file where hooks emit structured, typed events… type field uses a dot-separated topic hierarchy (e.g., algorithm.phase, work.created)." — `PAI/MEMORYSYSTEM.md`
- **EXECUTE** — *Doctrine* — Algorithm phase 5/7: do the work; "as each criterion passes, IMMEDIATELY edit ISA: [ ] → [x], update progress:" under the Inline Verification Mandate. — `PAI/Algorithm/` (LATEST)

## F

- **Fabric** — *Skill* — "Intelligent prompt pattern system with 240+ specialized patterns for content analysis, extraction, and transformation." Reference doc: `PAI/THEFABRICSYSTEM.md`. — `skills/Fabric/SKILL.md`
- **Fabrication** — *Doctrine* — "Generating a confident assertion that is not grounded in a source verified this session." The rule: "retrieve from a verified source, or do not assert." Enforced by the honesty layers (IdentityPin, IdentityValidator, ClaimAttributionScan, shims). — `PAI/PAI_SYSTEM_PROMPT.md`
- **fabricated-path** — *Doctrine* — HonestyValidator finding kind targeting "backtick-absolute and 'at/in/file /path' forms" — a path claimed in output that no tool call touched. Currently warn-only. — `PAI/ENFORCEMENT_MAP.md`
- **Fast-path mode** — *Doctrine* — "Available at Standard tier (E1) only. Compresses phases for simple tasks… a whitelist, not a heuristic. Compress to: OBSERVE → EXECUTE → VERIFY." — `PAI/Algorithm/mode-detection.md`
- **Feedback Memory** — *Memory* — `feedback_*.md` files (auto-memory) holding corrections/confirmed approaches. Consulted twice by doctrine: design-time at OBSERVE Step 0(3), pre-action at the PLAN Feedback Memory Auto-Consult. — `PAI/Algorithm/` (LATEST)
- **Forge** — *Codename* — "GPT-5.4 via codex exec, reasoning_effort=high — auto-include at E3/E4/E5 for any coding task (implement, refactor, debug, build, migration, fix, feature)." — `PAI/Algorithm/` (LATEST)
- **Functions API** — *Component* — "HTTP gateway on localhost:8890. No Claude API tokens required. Authenticated calls should use a signed PAI identity session." Systemd unit `pai-functions`; also the JIT credential gateway. — `PAI/FUNCTIONS_API.md`

## G

- **GAP** — *Doctrine* — Enforcement Map status: "a mechanism exists for one or more engines but not the full set named in the rule's scope." — `PAI/ENFORCEMENT_MAP.md`
- **Granularity Test** — *Doctrine* — "Split until each criterion is one binary tool probe… If you cannot name the probe, the criterion is not yet atomic — split it." The pre-THINK exit condition for ISC quality. — `PAI/Algorithm/` (LATEST)

## H

- **Hard-to-Vary Explanation** — *Core Concept* — Deutsch epistemology at PAI's root: "a description of reality (or of a goal) where every detail plays a functional role, so contrary evidence has nowhere to flee." — `PAI/DOCUMENTATION/PAISystemPhilosophy.md`
- **Herdr** — *Codename/Tool* — Multi-engine agent/pane/workspace manager; PAI drives it via `HerdrCockpit.ts` (`hc`): "status / launch / handoff / council (fan-out a prompt to N engines in exec mode)" over a direct Unix socket. — `PAI/Tools/HerdrCockpit.ts`
- **Honesty Floor** — *Doctrine* — "Three permitted forms for any technical or identity claim: Sourced… Hedged… Unknown — 'I don't know' is a valid answer. It always beats a fabrication." Overridden for safety-asserting claims by Security Escalation. — `PAI/PAI_SYSTEM_PROMPT.md`
- **HonestyValidator** — *Tool* — "Single source of truth for honesty/no-fabrication checks… validates that an output emitted by ANY PAI engine does not fabricate the DA's identity and does not contain confident unsourced claims." Called by IdentityValidator (block-mode for identity-mismatch) and ClaimAttributionScan (warn). Kill switch: `PAI_HONESTY_BYPASS=1` (logged). — `PAI/Tools/HonestyValidator.ts`
- **Hook** — *Component* — "TypeScript/Python scripts that run automatically when specific events occur in Claude Code sessions" (SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, Stop, SubagentStop, PreCompact…). Registered in `settings.json`; live in `hooks/`. — `PAI/ARCHITECTURE_RELATIONSHIPS.md`, `PAI/THEHOOKSYSTEM.md`
- **HookHealer** — *Hook* — SessionStart self-healing sweep: parses settings-registered hook commands and repairs missing exec bits on direct-exec scripts under `~/.claude` (realpath containment; warn-only, fail-open). Adopted 2026-07-05 from the LifeOS 6.0 delta-mine (ADOPT #2). — `hooks/HookHealer.hook.ts`
- **Human 3.0** — *Core Concept* — "The maturity arc PAI is built to move the Principal along": Aware → Activated → Aligned → Actualized. — `PAI/DOCUMENTATION/PAISystemPhilosophy.md`

## I

- **ICM** — *Memory* — Infinite Context Memory: MCP server for long-term memory across sessions (recall/store with topic + importance levels; memoirs; feedback). Surfaced by the `/recall` skill with EntityGraph boost. — icm MCP server, `commands/recall.md`
- **Ideate mode** — *Doctrine* — Algorithm mode (`ideate`/`id8` triggers): idea-generation loop configured by `ideate-loop.md`; sets `mode: ideate` in ISA frontmatter. — `PAI/Algorithm/mode-detection.md`
- **Identity Tokens** — *Doctrine* — "Your own name, the principal's name, the names of system components — must be retrieved from their canonical source, not generated from prior." Canonical source: `settings.json → daidentity.name` / `principal.name`. — `PAI/PAI_SYSTEM_PROMPT.md`
- **identity-mismatch** — *Doctrine* — The HonestyValidator finding kind in BLOCK mode: the Stop-hook emits `{decision: block}` JSON forcing regeneration when output uses a wrong DA/principal name. — `PAI/ENFORCEMENT_MAP.md`
- **IdentityPin** — *Hook* — "Emits an explicit identity-pin block at every session start… tells the model: 'your name is X; do not synthesize it.'" Honesty enforcement layer 1. — `hooks/IdentityPin.hook.ts`
- **IdentityValidator** — *Hook* — "Catches DA-identity fabrication at Stop/SubagentStop… scans the response text for fabricated DA-identity tokens — names that don't match settings.json → daidentity.name." Honesty enforcement layer 2 (block-mode for identity-mismatch). — `hooks/IdentityValidator.hook.ts`
- **Inference.ts** — *Tool* — "Unified inference tool with capability-matrix routing… via inference-matrix.json, task-based model selection, cost tracking integration." The dispatcher behind OllamaSkill task profiles and `--mode advisor`. — `PAI/Tools/Inference.ts`
- **Inline Verification Mandate** — *Doctrine* — "No ISC criterion may transition [ ] → [x] without verification evidence captured in the same tool call block that claims it, or the immediately-following block." Forbidden language in place of evidence: "should work", "done" without tool evidence, "no errors" without the log. — `PAI/Algorithm/` (LATEST)
- **Intent Echo** — *Doctrine* — Mandatory first OBSERVE action: "restate the user's request in ONE sentence… This line anchors the entire Algorithm run." — `PAI/Algorithm/` (LATEST)
- **Interceptor** — *Skill/Component* — "The ONLY sanctioned browser automation in PAI — real Chrome, no CDP detection, real login sessions, accurate rendering." Mandatory for web-output verification and deploy confirmation. — `PAI/PAI_SYSTEM_PROMPT.md`, `skills/Interceptor/SKILL.md`
- **ISA** — *Core Concept* — Ideal State Artifact: "the single document that articulates, drives, and verifies a thing's ideal state as a hard-to-vary explanation." Lives at `MEMORY/WORK/{slug}/ISA.md`; system of record for Algorithm runs ("The AI writes ALL content directly. Hooks only read."). Frontmatter: task/slug/effort/phase/progress/mode/started/updated. — `skills/ISA/SKILL.md`, `PAI/Algorithm/` (LATEST)
- **ISC** — *Core Concept* — Ideal State Criteria: "the irreducible, independently verifiable structure of 'done.' A vague checklist is not an ISC." Written as `- [ ] ISC-N: criterion`, one binary tool probe each. — `PAI/DOCUMENTATION/PAISystemPhilosophy.md`
- **ISASync** — *Hook* — "Read-only ISA → work.json sync via PostToolUse… ONLY reads the ISA and syncs to work.json for the dashboard." The ISA frontmatter `phase:` edit IS the phase signal. — `hooks/ISASync.hook.ts`
- **Isolation Gate** — *Doctrine* — PLAN gate for parallel write-agents: "Overlapping file targets → isolation: 'worktree'. Non-overlapping → skip. Read-only agents → never need worktree. Competing approaches → always worktree." — `PAI/Algorithm/` (LATEST)

## K

- **Kai incident** — *Doctrine* — The 2026-05-26 originating identity-fabrication failure (DA name "Kai" emitted from prior). Named failure modes: salience gap, plausibility hijack, template pressure, missing structural guard — the reason the honesty enforcement layers exist. — `PAI/PAI_SYSTEM_PROMPT.md`
- **KnowledgeHarvester** — *Tool* — "Knowledge graph maintenance tool… for each domain in PAI/MEMORY/KNOWLEDGE/{Ideas,People,Companies,Research}, generate _MOC.md" (Map of Content indexes). — `PAI/Tools/KnowledgeHarvester.ts`

## L

- **LEARN** — *Doctrine* — Algorithm phase 7/7: learning inventory + Learning Router, doc sync, reflection JSONL, then the SUMMARY block. — `PAI/Algorithm/` (LATEST)
- **Learning Router** — *Doctrine* — "Every 'should I remember this?' question goes through this single router" — routes each learning to its PAI surface (knowledge/rule/gotcha/state/business/identity/doctrine/hook/permission/reflection). "Default disposition: SKIP." — `PAI/Algorithm/` (LATEST)
- **Legion** — *Infra/Codename* — The laptop (`pai3`); hosts the Legion Voicebox Sound Server (port 17493, `POST /speak`) — the active Pulse voice backend — and the migrated Plane instance (:8091). — `PAI/ASSET_REGISTRY.md`
- **Life OS** — *Core Concept* — See PAI. "A Life Operating System — not just a chatbot, but a system that helps you run your life with persistent memory, delegation, coordination, and verification." — `PAI/ARCHITECTURE_RELATIONSHIPS.md`
- **loadAtStartup** — *Convention* — settings.json list of "files force-loaded into session context at startup by LoadContext.hook.ts… injected as system-reminder blocks." Paired with `postCompactRestore.fullFiles` for compaction survival. — `settings.json`
- **LoopDetector** — *Hook* — PostToolUse all-tools watcher: per-session sliding window of tool-call signatures; warns on 4 identical calls in 12 or 3 consecutive identical failures (cooldown-limited, warn-only, fail-open). Adopted 2026-07-05 from the LifeOS 6.0 delta-mine (ADOPT #3). — `hooks/LoopDetector.hook.ts`
- **LEARNING/ (REFLECTIONS · SIGNALS · FAILURES · SYNTHESIS)** — *Memory* — The learning surfaces: `algorithm-reflections.jsonl` (per-run LEARN reflections), `ratings.jsonl` (satisfaction signals), full-context failure dumps for low-sentiment events, and synthesized patterns. — `PAI/MEMORYSYSTEM.md`

## M

- **MEMORY/** — *Memory* — The unified memory tree (Memory System v7.0, "Projects-native architecture"): `WORK/` (per-task ISA/PRD dirs — primary work tracking), `KNOWLEDGE/` (People/Companies/Ideas/Research with typed cross-links), `STATE/` (ephemeral operational state, "optimized for speed, not permanence"), `LEARNING/` (see above). Claude Code's `projects/` transcripts are "the actual firehose." — `PAI/MEMORYSYSTEM.md`
- **Mercury** — *Codename* — "Mercury 2 fill-in-the-middle code completion via Inception Labs API" (MercuryFIM skill). — `skills/MercuryFIM/SKILL.md`
- **MetaMCP** — *Infra* — MCP proxy on port 3010 ("8 MCP servers proxied"). — `PAI/ASSET_REGISTRY.md`
- **MINIMAL mode** — *Convention* — Output mode for "greetings, ratings, single-token acknowledgments." — `PAI/PAI_SYSTEM_PROMPT.md`
- **Modes (MINIMAL / NATIVE / ALGORITHM)** — *Convention* — The three mandatory output formats; "decided by a deterministic classifier at UserPromptSubmit" (PromptProcessing) writing a `MODE:`/`TIER:` line. Every response uses exactly one. — `PAI/PAI_SYSTEM_PROMPT.md`, `CLAUDE.md`

## N

- **Named Agents** — *Component* — "Persistent identities with backstories and ElevenLabs voices (Serena, Marcus, Rook, etc.) — recurring work, voice output, relationships." — `PAI/PAIAGENTSYSTEM.md`
- **NATIVE mode** — *Convention* — Output mode for "a single fact lookup, a single-line edit on a named file, or one command run — AND no new artifact is created — AND no multi-step plan is required." All subagents use NATIVE. — `PAI/PAI_SYSTEM_PROMPT.md`
- **Nova** — *Codename* — Duane's DA (`daidentity.name: Nova`, aka PAI Nova; the identity behind AGT-001/PNC). — `settings.json`
- **/notify (port 31337)** — *Component* — `pai-pulse`: "canonical engine-facing voice compatibility route: /notify; forwards to Legion Voicebox." The curl target for Algorithm voice announcements. — `PAI/ASSET_REGISTRY.md`

## O

- **OBSERVE** — *Doctrine* — Algorithm phase 1/7: Intent Echo, Reverse Engineering, Preflight Gates A–F, Step 0 batch/parallelize + prior-art consult, Reproduce-First, effort setting, capability selection, ISC writing. — `PAI/Algorithm/` (LATEST)
- **OBSERVE Step 0** — *Doctrine* — "Before deep reads, in ONE round: enumerate every file the task will likely touch and Read them together; launch every independent background job up front… (3) consult prior art and probe the grammar — in OBSERVE, not at PLAN/THINK/BUILD." — `PAI/Algorithm/` (LATEST)
- **OllamaSkill** — *Skill* — "Local-first inference dispatch via PAI/Tools/Inference.ts — routes to Ollama for cheap task types… with automatic cloud fallback per inference-matrix.json." Task profiles: fast_classification (llama3.2:3b), structured_json (qwen2.5:7b), summarization (gemma2:9b), multi_step_reasoning (deepseek-r1:7b), code_generation (honestly routed to claude/sonnet). — `skills/OllamaSkill/SKILL.md`
- **OmniPulse** — *Codename* — Cross-engine dispatch/handoff layer; handoff artifacts under `~/MEMORY/STATE.claude/omnipulse-handoffs/`. Its honesty-validation role (layer 5) is NOT YET LIVE. — `PAI/PAI_SYSTEM_PROMPT.md` (layer table)
- **opai** — *Codename* — "OpenCode (opai) | AGT-003 | Separate CLI tool using Zen free models | Zero-cost coding tasks, budget-critical periods." The engine PNK runs. — `PAI/ARCHITECTURE_RELATIONSHIPS.md`
- **Operator** — *Org* — Archetype (aka SysAdmin, DBA, Release Manager): "Execute runbook-scoped operations on assigned assets/tickets. Destructive or undocumented actions require approval." — `PAI/RBAC_ORG_STRUCTURES_MODEL.md`
- **Optimize mode** — *Doctrine* — Algorithm mode (`optimize [target]`): measured improvement loop per `optimize-loop.md` (eval_mode metric vs eval). — `PAI/Algorithm/mode-detection.md`
- **ORA** — *Org* — Org Role Assignment: "Binds one UID to one org role in one org session" (ORA-{id}). Permissions require an active scoped ORA plus RBAC allow. — `PAI/RBAC_ORG_STRUCTURES_MODEL.md`

## P

- **PAI** — *Core Concept* — "Personal AI Infrastructure = the Life Operating System. It turns AI from a chatbot you talk to into a system that helps you run your life." Primary directive: understand the Principal to move them from current state to ideal state. — `PAI/DOCUMENTATION/PAISystemPhilosophy.md`
- **pai-core.ts** — *Tool* — "Phase-1 substrate kernel. The ONLY module allowed to know real PAI paths… resolves the PAI CONTENT root (Option B) explicitly and forward-compatibly." Use `memoryDir()` etc., never raw env. — `PAI/Tools/lib/pai-core.ts`
- **pai-guard.ts shims** — *Security* — PATH shims at `PAI/bin/<tool>` enforcing the destructive-pattern registry outside Claude Code; "shims fail-open on registry errors so the user's shell is never broken." Shell-tier override key: `shell:<ppid>:<tty>`. — `PAI/DOCUMENTATION/SECURITY_OVERRIDE_MECHANISM.md`
- **pai-infra.ts** — *Tool* — "Single source of truth for KNOWN infrastructure/environment… nothing about known infra may be RECALLED from memory. It is RESOLVED from this registry… it never guesses." The tool behind "resolve, don't recall." — `PAI/Tools/pai-infra.ts`
- **pai-primary** — *Infra* — "Primary AI/service hub (VM on aorus physical)" — the main PAI host (Proxmox VM100). — `PAI/ASSET_REGISTRY.md`
- **pai-prism** — *Infra* — "Kitchen Prism family kiosk (Linux appliance)… Chrome --kiosk via Xorg autologin." The family-facing display surface (calendar, dashboards, 3D home twin). — `PAI/ASSET_REGISTRY.md`
- **PAI_HONESTY_BYPASS** — *Security* — Env kill switch that "disables the [honesty] validator with a logged warning. Intended for hook-debugging only"; uses are logged to `honesty-bypass.jsonl`. — `PAI/PAI_SYSTEM_PROMPT.md`
- **Persona** — *Component* — "Rich narrative identities for agents — who they are, how they think, their voice, their style." Layered on archetypes/agents; not an authority grant. — `PAI/ARCHITECTURE_RELATIONSHIPS.md`
- **PhaseTransitionGate** — *Hook* — "Structural enforcement of Algorithm Rule 2. Blocks ISA edits that transition phase: to build unless the ISA Decisions section contains an advisor marker or a complete Rule 2 emission block." Fires at every PLAN→BUILD regardless of tier. — `hooks/PhaseTransitionGate.hook.ts`
- **PLAN** — *Doctrine* — Algorithm phase 3/7: feedback auto-consult, planning block, Deliverable Manifest, delegation/parallelism/async/watchdog/isolation/coordination/quota gates, then the PLAN→BUILD Advisor Gate. — `PAI/Algorithm/` (LATEST)
- **PNC / PNK / PNX / PNG / PNO** — *Codename* — The engine legend: "PNC — Claude Code (Anthropic, primary)… PNK — opencode… PNX — codex… PNG — antigravity / agy (Google)… PNO — Ollama local models (via PAI/Tools/Inference.ts)." Colloquially "PNK is Codex" means "a code agent" — PNK runs opencode, PNX runs codex. — `PAI/ENFORCEMENT_MAP.md`
- **PRD** — *Memory* — Product Requirements Document — the ISA's format lineage: "the single source of truth for every Algorithm run" (`PRDFORMAT.md` v2.0 still defines the ISA file spec; Algorithm renamed the artifact ISA). — `PAI/PRDFORMAT.md`
- **Preflight Gates (A–F)** — *Doctrine* — OBSERVE gates fired by task shape: **A Diagnostic** (reproduce before code archaeology; probe the authoritative dynamic source before any infra-attribution claim), **B Deploy/API** (credentials/CLI/service access exist), **C External service** (load skill context + gotchas), **D Research** (external docs before local archaeology), **E Existence Probe** (Glob/Grep for prior art or `confirmed-greenfield`), **F Prerequisite Probe** (empirically probe writability/tool-on-PATH/host-reachable/port-conflict before committing a write/start/call ISC). — `PAI/Algorithm/` (LATEST)
- **Principal** — *Org/Core Concept* — The human PAI serves (Duane, USR-001): "only principal accepts risk, spend, destructive ops, and authorization expansions." Canonical name: `settings.json → principal.name`. — `PAI/RBAC_ORG_STRUCTURES_MODEL.md`, `settings.json`
- **Producer** — *Org* — Archetype (aka Engineer, Volunteer, SRE, Toolsmith): "Update owned tasks and write work evidence. Cannot approve own work." — `PAI/RBAC_ORG_STRUCTURES_MODEL.md`
- **PromptProcessing** — *Hook* — The UserPromptSubmit dispatcher: deterministic MODE/TIER classifier ("regex/heuristic rules… instant, no model call") plus tab-title/session-name inference. Its `MODE:`/`TIER:` line is authoritative; absence = hook failure to flag. — `hooks/PromptProcessing.hook.ts`, `PAI/PAI_SYSTEM_PROMPT.md`
- **Proxmox (prox)** — *Infra* — The hypervisor on aorus (PVE 9.x, `ssh prox`); hosts pai-primary (VM100) and the HAOS VM; nightly vzdump backups. — `PAI/ASSET_REGISTRY.md`
- **Pulse** — *Component* — "The Life Dashboard: the visible surface onto the Life OS (http://localhost:31338)." Also the name of the voice-notify service lineage (`pai-pulse`, port 31337). — `PAI/DOCUMENTATION/PAISystemPhilosophy.md`

## Q

- **QNAP TS-464 (qnap-nas / NAS6DB92A)** — *Infra* — "Primary NAS — NFS /share/PAI → /PAI + /mnt/pai on pai-primary" (transport now SSH-only per the storage spine; off tailnet 2026-06-18). Hosts Shared-Inbox handoffs and backups. — `PAI/ASSET_REGISTRY.md`
- **Quota-Check Gate** — *Doctrine* — "Before spawning ≥2 parallel agents OR any Agent call with model:'opus', probe subscription headroom" via UsageProbe; <20% headroom → downshift to local/single-agent; <10% → defer or route local. — `PAI/Algorithm/` (LATEST)

## R

- **RBAC (two-layered)** — *Org* — "1. Identity RBAC: persistent trust tier and baseline capabilities for a UID. 2. Org-Scoped RBAC: temporary role grants inside a structure/team/session/resource boundary." Effective access = intersection. — `PAI/RBAC_ORG_STRUCTURES_MODEL.md`
- **recall / /recall** — *Memory* — "Hybrid ICM memory recall with EntityGraph proximity boost. Searches ICM memories using FTS5 + entity graph scoring." Read-side unification across ICM + auto-memory corpora. — `commands/recall.md`
- **refined:** — *Doctrine* — The ISA `## Decisions` prefix logging living-document refinements; counted as `living_doc_refinements` in the reflection JSONL. — `PAI/Algorithm/` (LATEST)
- **Reflection JSONL** — *Doctrine* — The per-run LEARN record appended to `MEMORY/LEARNING/REFLECTIONS/algorithm-reflections.jsonl` (tier, ISC counts, doctrine_fired flags, satisfaction prediction). Mined by `/pai-upgrade mine-reflections` to evolve doctrine. — `PAI/Algorithm/` (LATEST)
- **Re-Read Check** — *Doctrine* — "Final gate before LEARN… re-read the user's last message verbatim and enumerate every explicit ask against what actually shipped." Any ✗ blocks `phase: complete` and loops back to PLAN. — `PAI/Algorithm/` (LATEST)
- **Reproduce-First** — *Doctrine* — "If Preflight Gate A fired, a reproduction MUST be captured before ANY Read/Grep targets the suspect code path." Code analysis without reproduction is speculation. — `PAI/Algorithm/` (LATEST)
- **Resolve, don't recall** — *Convention* — "Any fact about known infra/environment MUST come from `bun PAI/Tools/pai-infra.ts resolve <thing>` or a live probe — NEVER from memory/plausibility." Read-path twin of the pai-provenance commit gate. — `CLAUDE.md`
- **Retrieve, don't generate** — *Doctrine* — "The retrieval/generation boundary is the rule: retrieve from a verified source, or do not assert." — `PAI/PAI_SYSTEM_PROMPT.md`
- **Reverse Engineering** — *Doctrine* — OBSERVE block enumerating explicit wants, explicit not-wanted, implied not-wanted, and the speed/urgency signal. — `PAI/Algorithm/` (LATEST)
- **Root-Cause-at-Ingestion Checkpoint** — *Doctrine* — BUILD checkpoint: "Where does this bad state enter the system?… If I fix it at the ingestion point instead of here, do 3 similar bugs disappear? If yes → move the fix upstream." — `PAI/Algorithm/` (LATEST)
- **RTK** — *Tool* — Rust Token Killer: "token-optimized CLI proxy (60–90% savings on dev operations)"; a hook transparently rewrites commands (`git status` → `rtk git status`). Meta commands: `rtk gain`, `rtk discover`, `rtk proxy`. — `RTK.md`
- **Rule 1 / 1b / 2 / 2a / 3** — *Doctrine* — The Verification Doctrine: **1 Live-Probe** (user-facing artifacts pass only with tool-verified probe evidence; persistent services must be running AND enabled), **1b Independent Cross-Check** (derived *sets* need two independent implementations asserting identity, not count), **2 Commitment-Boundary Advisor** (advisor() at PLAN→BUILD for E2+, when stuck, and before `phase: complete`), **2a Cross-Vendor Audit** (Cato at E4/E5), **3 Conflict-Surfacing** (never silently override the advisor; max two re-calls, then escalate to the user). — `PAI/Algorithm/` (LATEST)

## S

- **Security (archetype)** — *Org* — Aka CISO, SOC, Red Team, GRC: "Triage, scan authorized scope, preserve evidence, respond. Cannot accept risk unless T0." — `PAI/RBAC_ORG_STRUCTURES_MODEL.md`
- **Security Escalation** — *Doctrine* — For safety-asserting claims ("no vulnerabilities", "patched", "safe to ship") the Honesty floor's silent-drop exit is REMOVED: uncited = `UNVERIFIED` + open finding; "absence of a finding in a domain you were tasked to assess is itself UNVERIFIED unless you can cite the check." — `PAI/Algorithm/` (LATEST)
- **Security Override Protocol** — *Security* — The three-option flow (`override once` / `override session` / `override permanent`) when a tool call is blocked; two variants — path-based (`path` field) and pattern-based (`pattern_slug`, DestructiveOpGuard). Pending entries: `~/MEMORY/STATE.claude/security-pending.json`. — `CLAUDE.md`, `PAI/DOCUMENTATION/SECURITY_OVERRIDE_MECHANISM.md`
- **Security philosophy** — *Security* — "Block catastrophic, confirm destructive, alert suspicious, allow everything else." Bash tiers: trusted (fast-path allow) / blocked (exit 2 catastrophic) / confirm / alert. Path zones: zeroAccess (secrets), readOnly, confirmWrite, noDelete. Without a patterns file SecurityValidator is fail-open. — `PAI/PAISECURITYSYSTEM/patterns.example.yaml`
- **Self-Healing Infrastructure** — *Doctrine* — "When the system fails… fix the system, not your notes. An OS doesn't accumulate sticky notes about its own bugs, it patches itself." Rules go to CLAUDE.md/hooks/settings/skills — never memo files. — `PAI/PAI_SYSTEM_PROMPT.md`
- **SessionBus** — *Tool* — "Cross-session/cross-engine coordination for parallel PAI sessions… EMIT / SURFACE / MUTEX (flock on a shared lockfile so concurrent ~/.claude committers can't clobber HEAD)." Use `SessionBus commit` for concurrent-session commits. — `PAI/Tools/SessionBus.ts`
- **Skill** — *Component* — "Self-contained instruction sets that teach the DA how to perform specific domains of work" (`skills/<Name>/SKILL.md` + `Workflows/`). The Skill System is "the MANDATORY configuration system for ALL PAI skills." ~127 skill directories at writing. — `PAI/ARCHITECTURE_RELATIONSHIPS.md`, `PAI/SKILLSYSTEM.md`
- **Spine** — *Component/Tool* — The Harness Spine thin kernel (`PAI/Tools/Spine/`): "migrate / derive / seed-enforcement / render / doctor. Freshness invariant: render/check ALWAYS derive first; the DB is a cache, not a source of truth." Emits `SPINE:GENERATED` markers into docs like HARNESS.md. — `PAI/Tools/Spine/cli.ts`
- **Splitting Test** — *Doctrine* — Split an ISC when: it joins two verifiable things ("and"/"with"), parts can fail independently, it uses scope words ("all/every/complete"), it crosses a domain boundary, or no single tool probe can verify it. — `PAI/Algorithm/` (LATEST)
- **STATE.claude** — *Memory* — The doctrine path `~/MEMORY/STATE.claude/` for engine-adjacent runtime state (security-pending.json, honesty-ramp.json, usage-canonical-cache.json, omnipulse-handoffs/…), kept outside the root-owned legacy tree. — `PAI/DOCUMENTATION/SECURITY_OVERRIDE_MECHANISM.md`
- **SUMMARY block (7/7)** — *Doctrine* — The mandatory Algorithm closing: "the ONLY acceptable final output is this block" — 🔄 ITERATION / 📃 CONTENT / 🖊️ STORY / 🗣️ signature. Prose recaps after it are a critical failure. — `PAI/Algorithm/` (LATEST)
- **Switchboard** — *Component* — AGT-004: "file-based message routing between engines / real-time message routing between agents." — `PAI/ARCHITECTURE_RELATIONSHIPS.md`

## T

- **Telos** — *Core Concept/Skill* — "The goal system: the Principal's mission, goals, metrics, challenges, and strategies that define where 'ideal state' actually points." Files under `PAI/USER/TELOS/`; the Telos skill analyzes them. — `PAI/DOCUMENTATION/PAISystemPhilosophy.md`
- **THINK** — *Doctrine* — Algorithm phase 2/7: riskiest assumptions, premortem, prerequisites, ISC refinement, euphoric-surprise prediction, Under-Claim Check, Security Escalation. — `PAI/Algorithm/` (LATEST)
- **Troubleshooting Method / 5-Step Loop** — *Doctrine* — "Troubleshooting is an iterative loop, not a linear path… a session that shows only the final success is hiding the method." The loop: show failing state → state hypothesis → try ONE fix → re-probe → iterate/escalate (FIXED/SAME/WORSE/DIFFERENT). E2+ debugging ISAs require a `## Troubleshooting Log`. — `PAI/DOCUMENTATION/TROUBLESHOOTING_METHOD.md`
- **Trust, but Verify** — *Doctrine* — "Extend good faith freely… but never let trust stand in for evidence. Trust is the posture; verification is the discipline." — `PAI/PAI_SYSTEM_PROMPT.md`

## U

- **Under-Claim Check** — *Doctrine* — For components that make claims about their own behavior: "name the mechanical proof that substantiates it. If you cannot name the proof, under-claim — state only the weaker thing you can prove." — `PAI/Algorithm/` (LATEST)
- **unsourced-claim** — *Doctrine* — HonestyValidator finding kind (warn-only telemetry): a claimed path/fact with no corroborating tool call in the session log. — `PAI/ENFORCEMENT_MAP.md`
- **UsageProbe / Canonical Budget Source** — *Tool/Convention* — "Query Anthropic's canonical usage endpoint… the authoritative subscription window state, not a local estimate" (GET /api/oauth/usage, mandatory 5-min cache at `usage-canonical-cache.json`). The only legitimate budget input for the Quota-Check Gate. — `PAI/Tools/UsageProbe.ts`, `PAI/DOCUMENTATION/BUDGET_SOURCE.md`

## V

- **Verifier** — *Org* — Archetype (aka QA, Reviewer, Board, Assessor): "Read, challenge, block closure, certify outcome. Cannot perform producer changes in same gate." — `PAI/RBAC_ORG_STRUCTURES_MODEL.md`
- **VERIFY** — *Doctrine* — Algorithm phase 6/7: the Verification Doctrine (Rules 1/1b/2/2a/3), per-ISC evidence, capability/preflight/doctrine/deliverable compliance checks, and the Re-Read Check. — `PAI/Algorithm/` (LATEST)
- **Verification Doctrine** — *Doctrine* — "Four rules govern every VERIFY pass. They are NOT optional." See Rule 1/1b/2/2a/3. — `PAI/Algorithm/` (LATEST)
- **Voice Announcements** — *Doctrine/Component* — "At Algorithm entry and every phase transition, announce via direct inline curl" to the :31337 `/notify` route; always self-identifying ("Entering the <phase> phase for <the TASK>"). Voice is audio-only; the ISA edit is the phase signal. Voice ID from `MEMORY/STATE/voice-config.json`. — `PAI/Algorithm/` (LATEST)
- **VoiceServer / Voicebox** — *Component/Infra* — The voice runtime: local PAI Voice Server (port 8888, fallback) with Pulse (:31337) forwarding to the Legion Voicebox Sound Server (:17493, `POST /speak`) — "talk in, PAI answers aloud." — `PAI/VoiceServer/server.ts`, `PAI/ASSET_REGISTRY.md`

## W

- **Watchdog Gate** — *Doctrine* — "On first background agent spawn in a session, start the agent watchdog if not running" (`Monitor` + `PAI/Tools/AgentWatchdog.ts`). — `PAI/Algorithm/` (LATEST)
- **Wazuh** — *Infra* — The SIEM: Wazuh Manager (agent log ingestion, OSSEC protocol, port 1514) + Wazuh Dashboard (:8601) with indexer at :9200; manager runs on the NAS. — `PAI/ASSET_REGISTRY.md`

---

*Registered in `PAI/CONTEXT_ROUTING.md` (PAI System table) and `PAI/DOCUMENTATION/INDEX.md`. Companion surfaces: `PAI/USER/DEFINITIONS.md` (personal definitions), `PAI/DOCUMENTATION/QUICK_REFERENCE_CARD.md`, `PAI/DOCUMENTATION/NAVIGATION_MAP.md`.*
