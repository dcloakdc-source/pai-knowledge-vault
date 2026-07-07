# Algorithm Changelog

Each entry: version, date, what changed, provenance, migration/rollback notes.

---

## v5.7.11 — 2026-06-18

ISA Troubleshooting Log and cross-engine troubleshooting protocol. Additive — all gates, rules, and phases preserved verbatim from v5.7.10; only the ISA body section listing, a new Troubleshooting Method section, and the preamble changed.

- **ISA `## Troubleshooting Log` body section added.** The ISA body now recognizes `## Troubleshooting Log` as a fifth section (alongside Context, Criteria, Decisions, Verification). Required at E2+ when Gate A fires (debugging). Captures the iteration chain: probe → hypothesis → fix → re-probe for each attempt. Format per `PAI/DOCUMENTATION/TROUBLESHOOTING_METHOD.md`. E1/additive work skips it.
- **Cross-engine troubleshooting protocol referenced.** Algorithm now names `PAI/DOCUMENTATION/TROUBLESHOOTING_METHOD.md` as the canonical troubleshooting protocol. The 5-step loop (show failing state → state hypothesis → try one fix → re-probe → iterate) is defined there; the Algorithm defines when the log is required.
- **Engine configs updated in same change:** PNC `settings.json verbose: false → true`; PNX `.codex/AGENTS.md` gains troubleshooting section + `--verbose` default; PNK `.opencode/AGENTS.md` gains protocol reference; PNC `CLAUDE.md` gains cross-engine references section.

Provenance: Duane asked "how can we enable the other PAI engines to follow the method of thought and troubleshooting that opencode displays" (2026-06-18). Cross-engine gap analysis produced the three-layer solution (protocol doc + engine configs + Algorithm ISA log). Duane: "Proceed" on adding the Troubleshooting Log to the Algorithm. Migration: none (additive doctrine + additive engine config edits). Rollback: set `LATEST` → `v5.7.10`; revert engine config edits.

Mined candidates #1 + #3 from the 2026-06-16 reflection-mining run. Additive — all gates/rules/phases preserved verbatim from v5.7.9; only the Gate A row and a new BUILD checkpoint changed.

- **OBSERVE Gate A extended to infra-attribution (#1, dominant 20+-hit theme).** For any root-cause/attribution claim about a host/ACL/policy/mount/timing, probe the authoritative dynamic source (read the ACL policy, measure host load, `pai-infra.ts resolve`) before forming the hypothesis. Implemented as a Gate A extension, not a 7th gate (v5.7.9 advisor anti-proliferation guidance); cites-and-extends CLAUDE.md "resolve, don't recall" + `feedback_verify-gated-conditions-before-attribution` rather than restating. Distinct from Gate F (precondition existence). Provenance: guessed-ACL-from-ping-timeout; model-speed conclusion before measuring the load-34 host storm.
- **BUILD Backend-Falsification Checkpoint (#3, the s5 over-budget run).** Before building a pipeline that writes to a stateful/external backend, throwaway write+read-back on a disposable key first — confirm the write *method* persists. Provenance: a full auto-on-close profile-writer built before discovering `state.vscdb` is unreachable by file edits.

Companion same-selection changes outside this file: **#2** — `deadman-registry.json` gains an `omnipulse-digest-distribution` total-silence probe + an `_outcome_doc` (liveness-of-trigger ≠ liveness-of-outcome; per-channel outcome-liveness deferred as named follow-up — no failure-only artifact exists yet). **#4** — `PAI_SYSTEM_PROMPT.md` Operational Rule: grep all callers before claiming a shared-contract fix complete.

Provenance: Duane selected all four mined candidates (review ISA `20260616-085146`; mining via PAIUpgrade; build ISA `20260616-092545_apply-4-mined-candidates`). Migration: none (additive doctrine + additive registry/constitution edits). Rollback: set `LATEST` → `v5.7.9`; revert the `deadman-registry.json` and `PAI_SYSTEM_PROMPT.md` edits via their checkpoint SHAs.

## v5.7.9 — 2026-06-16

Design-time prior-art consult promoted into OBSERVE. Additive — all gates/rules/phases preserved verbatim from v5.7.8; only OBSERVE Step 0 (new item 3) and a one-line PLAN cross-ref changed.

- **OBSERVE Step 0 item (3): consult prior art + probe schema in OBSERVE, not later.** Mined from the 2026-06-16 reflection review — **12/55** unmined reflections (≈22%, the single dominant theme) cited a prior-art / feedback-memory / schema read that landed at PLAN/THINK/BUILD when it belonged in OBSERVE. Unconditional prior-art/root-cause consult (`MEMORY/KNOWLEDGE/` + `feedback_*.md` for design-bearing hits); conditional schema/tool-grammar probe (only when an ISC depends on one); relevance framed as search-then-consult. Differentiated from PLAN's `FEEDBACK MEMORY AUTO-CONSULT` by *purpose* (OBSERVE = design-time prior-art; PLAN = pre-action gotcha re-check on the chosen approach), with reciprocal cross-references so the boundary survives a real session. Harm anchors: a wrong NetworkManager DNS root cause the advisor had to overturn (feedback memory read at PLAN not OBSERVE); an unmeasurable FP-gate claim made before probing the stripped-`path` telemetry schema. Falsification: the next `/pai-upgrade mine-reflections` run measures recurrence/decay of this theme. Advisor verdict on the change: concerns (boundary under-specification + missing falsification loop) — both addressed before write. Provenance: Duane — "address findings" on the reflection review (sibling ISA `20260616-085146_review-reflections`; build ISA `20260616-085748_observe-prior-art-consult-v579`). Migration: none (additive doctrine). Rollback: set `LATEST` → `v5.7.8`.

## v5.7.8 — 2026-06-14

Self-identifying voice announcements. Additive — only voice-announcement wording changed; all gates/rules/phases preserved verbatim from v5.7.7.

- **Every Algorithm voice announcement states the task.** Entry is `"Entering the Algorithm for <the 8-word TASK>"` and each phase transition is `"Entering the <Phase> phase for <the TASK>"` (from the `🗒️ TASK` banner) instead of context-free phrasing; the bare phrasing is forbidden across entry and all seven phases (OBSERVE→LEARN). Provenance: Duane — "announcement for entering the algorithm should state what for" + "we need the same for all of the algorithm steps" (2026-06-14). Migration: none (wording only). Rollback: set `LATEST` → `v5.7.7`.
- **Rapid-phase coalescing (runtime, `VoiceServer/server.ts`).** Trailing-debounce: when phase transitions fire within ~5s of each other, only the last keeps the spoken task context (earlier ones bare). Agent always emits the full form; coalescing is VoiceServer-side. Provenance: Duane — "if the phases fall within 5 seconds of each other, only add the context to the last one."

## v5.7.7 — 2026-06-13

One additive block from the same 2026-06-10 mining run that produced Gate F + v5.7.6 (candidate #5). Byte-copy of v5.7.6 + title version + the Quota-Check Gate; every prior rule preserved.

- **PLAN-phase Quota-Check Gate** (mined #5, 7/272 ≈ 3%): before spawning ≥2 parallel agents OR any `Agent` call with `model:"opus"`, probe subscription headroom via `bun PAI/Tools/UsageProbe.ts`. Three-tier action (proceed / downshift / defer); inserted in PLAN after the COORDINATION GATE. Pairs structurally with the Local-First Routing Policy in CLAUDE.md — that policy says *prefer* local; this gate makes "before heavy spawn, *check*" the procedural step.

Provenance: cited 2026-03-06 reflection where the executor spawned expensive parallel agents and the user manually interrupted to redirect to Ollama. Migration: none. Rollback: set LATEST back to `v5.7.6`.

---

## v5.7.6 — 2026-06-12

Two additive blocks from the same 2026-06-10 mining run that produced Gate F (candidates #2 and #3). Byte-copy of v5.7.5 + title version + the two blocks; every prior rule (incl. Gate F) preserved.

- **OBSERVE Step 0 — Batch & Parallelize Inputs** (mined #2, 24/272 ≈ 9%): read all known target files together and launch independent background jobs in one parallel round before deep reads. Inserted in OBSERVE after the preflight output block.
- **Persistent-service completion gate** (mined #3): VERIFY Rule 1 artifact table gains a row — network-listener / long-running-service ISCs require a *running AND enabled* unit (`systemctl is-active` + `is-enabled`), never a foreground/test process.

Provenance: `/pai-upgrade mine-reflections` over 272 reflections (2026-06-10). Migration: none (additive). Rollback: set `PAI/Algorithm/LATEST` back to `v5.7.5`.

---

## v5.7.5 — 2026-06-10

OBSERVE Preflight **Gate F (Prerequisite Probe)** added — additive; byte-copied from v5.7.4 with only the title version line + the Gate F table row changed. For build/edit/deploy ISCs that write a path, start a service, or call an external tool/host, probe the precondition empirically (writability+ownership, tool-on-PATH, host/mount reachability, port/process conflict) before committing the ISC.

Provenance: `/pai-upgrade mine-reflections` over 272 algorithm reflections (2026-06-10) — Gate-F's theme was the single dominant pattern at 96/272 (35%, still 39% in recent runs). Canonical live case: the 2026-06-08 backup-infra session (NFS permission wall, 13-day-silent qdrant backup, three path-existence bugs — all Gate-F catches). The mining itself was only possible after fixing the reflection-pipeline PAI_DIR split (commit `fcd2e66`) that had left the store unreadable.

Migration: none (additive). Rollback: set `PAI/Algorithm/LATEST` back to `v5.7.4`.

### Addition — Gate F (Prerequisite Probe)

Inserted after Gate E in the OBSERVE preflight table. Complements Gate E (prior-art existence) and Gate B (credentials) without overlap — F is runtime *state*; absorbs the recurring permissions-gate pattern as sub-clause (a). Fires at all tiers including E1; a blank `🚦 F:` block when a write/start/call ISC exists is a doctrine violation.

---

## v5.7.4 — 2026-05-30

One additive security-escalation clause in the UNDER-CLAIM CHECK; byte-copied from v5.7.3 (only the title version line changed). Provenance: ISA `20260530-122027_source-citation-doctrine-v574` (11-model UnifiedCouncil honesty red-team `council-20260530120336-f6234f`).

### Addition — Security escalation (UNDER-CLAIM CHECK)

**What changed:** For *safety-asserting* claims (asserting the absence of a problem — "no vulns", "patched", "redacted", "enforced", "safe to ship"), the Honesty floor's silent-drop exit is REMOVED: an uncited safety claim is `UNVERIFIED` + an open finding; the absence of a finding in a tasked domain is itself `UNVERIFIED` unless the all-clear check is cited (anti-omission); resolution requires a citation, not a narrative. **Provenance:** the council (GLM/MiniMax) found v5.7.3's "just under-claim" wording prompt-injection-*weaponizable* (steer the model to silently suppress a true finding); DeepSeek/Kimi proposed the source-citation fix; advisor-hardened (anti-omission limb, directional scope, cited-resolution, softened absolute). **Override:** overrides "Confidence requires source" for the safety-asserting subclass only (removes the silent-drop option there).

**Migration/rollback:** Additive. `LATEST` flips to `v5.7.4` after the run; rollback = restore `LATEST` to `v5.7.3` (`v5.7.4.md` stays inert unless LATEST points to it).

---

## v5.7.3 — 2026-05-30

Three additive rules promoted from battle-tested Spine patterns; byte-copied from v5.7.2 (zero rule deletions). Provenance: ISA `20260530-034955_spine-council-recommendations` (Spine council post-session review).

### Addition 1 — Independent Cross-Check (VERIFY, Rule 1b)

**What changed:** New VERIFY sub-rule. When a step extracts/derives a *set* from sources, run two genuinely-independent implementations and assert set *identity* (not just count). **Provenance:** Spine `hooks-extract.ts` A-vs-B cross-check (PATH A structured parse vs PATH B raw regex) — caught a real UserPromptSubmit under-model. **Trigger:** any VERIFY step where a wrong set would pass an ordinary live-probe.

### Addition 2 — Under-claim deliberately (THINK design-review question)

**What changed:** New THINK design-review question. For every claim a component makes about its own behavior, name the mechanical proof; if you can't, under-claim. **Provenance:** Spine `probe.ts` ("never prints enforced/verified"). **Trigger:** building validators/probes/guards/registries/health surfaces.

### Addition 3 — E5 Session-Scope Checkpoint (conditional, event-triggered)

**What changed:** New conditional gate, E5 only. At a phase boundary, IF context budget high OR phase crossed a large tool-call count, ask once whether the next phase should wait for a fresh session; write a one-line Decisions note ONLY when the answer is "wait" (silent otherwise). **Provenance:** Spine E5 run `20260529-182417` (P3-inc2 deferral). Advisor-shaped to be event-triggered + silent-on-no, not a fixed cadence — avoids ceremony tax.

**Migration/rollback:** Additive only. `Algorithm/LATEST` flips to `v5.7.3` after the run's Cato gate passes (cutover-last). Rollback = restore `LATEST` to `v5.7.2`; `v5.7.3.md` can remain (inert unless `LATEST` points to it).

---

## v5.7.1 — 2026-05-09

### Patch 1 — Preflight Gate E (Existence Probe)

**What changed:** Added a fifth row to the OBSERVE preflight gates table.

**Trigger:** Build / add / create / implement verbs.

**Goal:** Run a single Glob or Grep against the natural target directory before THINK commits. Either find prior art (incorporate into ISCs) or confirm greenfield (document `confirmed-greenfield: <reason>` in `## Decisions`).

**Tier gate:** ALL tiers including E1.

**Provenance:** 3× recurring reflection pattern across `hook-observability-schema`, `memory-acl`, `audit-gaps` (2026-05-03). Existing four gates (A/B/C/D) all reported N/A on greenfield-on-existing-scaffold and hardening work, leaving the most-needed coverage silent.

**Verification target:** Reflection mining 30 days post-release should show ≥50% drop in "should have probed earlier in OBSERVE" Q1 entries.

**Rollback:** Delete the new table row in `v5.7.1.md` preflight gates section. Doctrine reverts to v5.7.0 four-gate behavior.

### Patch 2 — Rule 2 PLAN→BUILD Hard-Binding

**What changed:** Rule 2 commitment-boundary advisor call upgraded from soft guidance to a phase-transition gate. At E2+ effort, the voice announcement `"Entering the Build phase"` MUST NOT fire until `advisor()` returns. Added a `🛡️ PLAN→BUILD ADVISOR GATE` block to PLAN phase as the operational surface.

**Skip-clause hardening:** Skip is invalid if PLAN produced a Deliverable Manifest with 2+ entries OR ISC count crossed the E2 floor (16). Wall-clock + file-count alone (v5.7.0) missed cases where PLAN named 3 deliverables but work happened to fit in 4 minutes. Manifest size and ISC count are PLAN-phase signals — they predict commitment scope before BUILD begins, which is the right input for a PLAN→BUILD gate.

**Tier gate:** E2+ (E1 fast-path skips Rule 2 entirely).

**Provenance:** 2× direct reflection pattern across `cross-engine-memory-v0` (2026-05-01) and `memory-acl` (2026-05-03). Prior soft "before BUILD begins" wording was empirically being deferred until VERIFY, by which point rework cost was high.

**Verification target:** Reflection mining 30 days post-release should show ≥50% drop in "should have invoked advisor earlier" Q1/Q2 entries.

**Rollback:** Restore the v5.7.0 Rule 2 wording in `## Verification Doctrine` and remove the `🛡️ PLAN→BUILD ADVISOR GATE` block from PLAN phase.

### Migration Notes

- No existing ISA needs migration. The new gate fires on new OBSERVE runs only.
- Existing in-flight Extended+ ISAs that have already passed PLAN may proceed without the new gate — applies to v5.7.1-onward sessions.
- Advisor call cost: ~30s wall-clock per PLAN→BUILD transition at E2+. Budget impact: negligible at E3+, noticeable at E2 (~3min budget).

### Files Changed

- `PAI/Algorithm/v5.7.1.md` (new — copy of v5.7.0 with two patches applied)
- `PAI/Algorithm/LATEST` (flipped from `v5.7.0` to `v5.7.1`)
- `PAI/Algorithm/changelog.md` (this file — created fresh; v5.7.0.md mentions a changelog but none existed on disk before now)

### Approval Trail

- Surfaced: 2026-05-08 via triple PAIUpgrade run (mine reflections + full upgrade)
- Proposed: 2026-05-08 in `MEMORY/WORK/20260508-232247_address-upgrade-items/ALGORITHM_V5_7_1_PROPOSAL.md`
- Approved by Duane: 2026-05-09 ("apply now")
- Applied: 2026-05-09
