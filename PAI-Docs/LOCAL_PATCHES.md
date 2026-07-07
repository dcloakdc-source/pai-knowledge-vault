# PAI Local Patches

Documenting non-upstream customizations and integration points for PAI tools.

---

## PATCH-1 — PlanChecker Integration (Algorithm PLAN phase)

**File:** `PAI/Tools/PlanChecker.ts`
**Integration point:** End of ALGORITHM MODE PLAN phase, before entering EXECUTE

After completing the PLAN phase (ISC criteria drafted, PRD written), run:

```bash
bun PAI/Tools/PlanChecker.ts
```

or via slash command: `/check-plan`

**What it does:** Sends the current PRD's ISC criteria to Ollama llama3.2:3b with a
structured scoring prompt. Returns SCORE/PASS/GAPS/VERDICT output.

**When to use:**
- Before marking phase: execute in the PRD frontmatter
- When ISC criteria feel underspecified or overlap is suspected
- As a sanity check before committing to a large EXECUTE workload

**Fail behaviour:** If Ollama is unreachable (exit 2), the check fails open — proceed
to EXECUTE normally. PlanChecker is advisory, not a hard gate.

**Pass threshold:** Score ≥ 7/10 + no CRITICAL gaps = proceed. Score < 7 or CRITICAL
gaps present = revise ISC criteria before entering EXECUTE.

---

## PATCH-2 — Seeds Forward-Planting (Algorithm EXECUTE phase)

**File:** `PAI/Tools/Seeds.ts`
**Integration point:** During and after EXECUTE phase

Plant forward-looking ideas during implementation:

```bash
bun PAI/Tools/Seeds.ts plant "Consider extracting X into shared lib" --milestone v4.1
```

or via slash command: `/seed plant "description"`

Seeds surface at session start via LoadContext banner if any are pending.
Use `/seed list` to review. `/seed done <id>` to close.

---

## PATCH-3 — Workstream Context (Algorithm session start)

**File:** `PAI/Tools/Workstreams.ts`
**Integration point:** LoadContext SessionStart banner

Active workstream name and branch appear in the LoadContext banner at session start.
Switch workstreams with `/workstream switch <id>` before starting Algorithm work to
ensure PRD context is scoped correctly.

---

## PATCH-4 — Model Profile (BudgetGuard thresholds)

**File:** `PAI/Tools/ModelProfile.ts`
**Integration point:** `hooks/BudgetGuard.hook.ts` reads `MEMORY/STATE/model-profile.json`

BudgetGuard uses WARN/CRITICAL thresholds from the active model profile:

| Profile  | WARN | CRITICAL | Ollama mode |
|----------|------|----------|-------------|
| quality  | 92%  | 99%      | off         |
| balanced | 80%  | 92%      | failover    |
| budget   | 60%  | 75%      | prefer      |

Set profile: `/profile set budget` (or `bun PAI/Tools/ModelProfile.ts set budget`)
View current: `/profile`
