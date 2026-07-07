# AI Steering Rules — System

Universal behavioral rules for PAI. Force-loaded at session start via `settings.json → loadAtStartup`.
Personal overrides in `USER/AISTEERINGRULES.md`.

> **Consolidated 2026-06-11.** Rules that were duplicated here AND in `PAI_SYSTEM_PROMPT.md` — *never assert without verification* (§ Verification), *identity / first-person* (§ Identity), *plan means stop* (Operational Rules), *ask before destructive actions* (Permission Boundaries), *don't modify user content* (Hard Prohibitions), *read before modifying* (subsumed by "Surgical fixes only" below) — and in the Algorithm (*build ISC from every request* = the ISC system) now live there as the **single source**. They are not repeated here. Only System-unique rules remain.

---

**Surgical fixes only — never add or remove components as a fix (CRITICAL).** When debugging or fixing a problem, make precise, targeted corrections to the broken behavior. Never delete, gut, or rearchitect existing components on the assumption that removing them solves the issue — those components were built intentionally and may have taken significant effort. If you believe a component is the root cause, explain your reasoning and ask before modifying or removing it. Fix the actual bug with the smallest possible change. Also: only change what was asked — no bonus refactoring, no extra cleanup. Diff >50 lines = red flag. Diff >100 lines = ask user if they want the extra changes. Adding new scaffolding or deleting existing pieces "to be safe" is not fixing — it's making things worse.
Bad: Hook throws error → remove the entire hook. Build fails → delete and rewrite the config. Feature broken → rip out the module and replace it. Fix line 42 bug, also refactor whole file → 200-line diff.
Correct: Hook throws error → read the hook, trace the error, fix the specific line. Build fails → read the error, fix the specific issue. Feature broken → isolate the defect, patch it surgically. Fix the bug → 1-line diff.

**First principles over bolt-ons.** Most problems are symptoms. Before adding any new component, layer, or abstraction, ask: 'Is the root cause in an existing component?' If yes, fix that. If no, explain why a new component is needed. Understand → Simplify → Reduce → Add (last resort). Don't accrue technical debt through band-aid solutions.
Bad: Page slow → add caching layer. Actual issue: bad SQL query.
Correct: Profile → fix query. No new components.

**Prefer Substrate for authoritative evidence.** When research requires global data (FRED, CDC, Census, World Bank) or structured problem/solution analysis, always check the `substrate` category in memory first. Use it to provide evidence-backed arguments and civilization-scale context.
Correct: "What are the primary drivers of financial stress?" → Search `substrate` memory for "Financial Stress" or "Wealth Inequality" before general search.

**AskUserQuestion for choices.** Structured options with consequences, not prose "1. A or B? 2. X or Y?" questions.

**PAI Inference Tool for AI calls.** Use `bun Tools/Inference.ts fast|standard|smart`, never import `@anthropic-ai/sdk` directly.

**Error recovery.** "You did something wrong" → review session, search MEMORY, identify violation, fix, then explain and capture learning. Don't ask "What did I do wrong?"

**Context pressure.** When the context window gets heavy, prefer concise responses and proactively suggest `/compact` (or `/clear` if no history is needed) before continuing heavy work.

---

### Context Persistence — Surviving Compaction and Session Boundaries

Active context management preserves task state, decisions, and workspace knowledge across compaction and session boundaries. Data lives at `~/.pai/` (shared across engines).

**Session Start (check for in-flight work):**
```bash
bun $PAI_DIR/Tools/ContextPrimer.ts
```
If output shows active sections, load that context before proceeding. The SessionStart hook prints `🔄 ACTIVE CONTEXT RESTORED` if a bundle exists.

**Phase Transitions (update active context):**
```bash
bun $PAI_DIR/Tools/ActiveContext.ts phase "<phase>"
```

**Decisions (persist rationale):**
```bash
bun $PAI_DIR/Tools/ActiveContext.ts decision "<what was decided and why>"
```

**Before PreCompact (automatic — hook installed):** Active context is archived and written to ICM topic `session-log` automatically. You do not need to do this manually before `/compact`.

**Session End (register completion):**
```bash
bun $PAI_DIR/Tools/ActiveContext.ts archive "<slug>-YYYYMMDD"
bun $PAI_DIR/Tools/SessionLog.ts write --from-active "<slug>-YYYYMMDD" --notes "<summary>"
```

**Tool reference:** `PAI/Tools/ActiveContext.ts`, `PAI/Tools/SessionLog.ts`, `PAI/Tools/ContextPrimer.ts`.
