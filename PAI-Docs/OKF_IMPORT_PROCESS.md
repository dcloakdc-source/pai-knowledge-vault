# Safe OKF Bundle Import Process

How external OKF bundles reach PAI recall **without poisoning memory**. External bundles are
untrusted content that future recall agents will read — the threat is *stored/memory injection*.
This process keeps external content quarantined, scanned, human-gated, and provenance-tagged.

## Trust tiers (never commingle)

| Tier | Path | Trust | Recall |
|------|------|-------|--------|
| **Quarantine** | `~/MEMORY/imported/okf/<b>/` | untrusted, injection-scanned | **NOT** recalled |
| **Reference** | `~/MEMORY/reference/okf/<b>/` | reviewed external, tagged `source: external` | recall-readable, **tagged external** |
| **Trusted** | `~/MEMORY/auto/` | Duane's OWN facts | recalled as authoritative |

**External content NEVER lands in `~/MEMORY/auto/`.** It is reference material, not Duane's assertion.

## Pipeline

```
discover → clone-quarantine → scan-import → DRY-RUN review → adversarial pass → APPROVE → promote → (recall)
```

1. **Discover** — `bun PAI/Tools/okf-discover.ts` (weekly timer `okf-discover.timer`). Ranks bundles vs tooling; dedupes.
2. **Clone to quarantine (read-only, hooks off):**
   `git -c core.hooksPath=/dev/null clone --depth 1 <repo> ~/MEMORY/imported/_quarantine/<name>`
   Never run the repo's code. Characterize size/file-count first (reject mega-dumps).
3. **Scan + import** — `bun PAI/Tools/okf-import.ts <src> <name>` → deterministic injection scan
   (override/role-hijack/meta-to-agent/exfil/credential/tool-injection/hidden-unicode), hidden-char
   sanitize, scripts flagged-not-run, external hosts listed-not-fetched. HIGH-severity files are HELD.
   Lands in the **quarantine** tier with a `_SECURITY_REPORT.md`.
4. **DRY-RUN review** — `bun PAI/Tools/okf-promote.ts <name>` (no `--approve`). Shows the gates.
5. **Adversarial pass (recommended for anything non-trivial)** — a second, independent opinion on
   whether the content tries to manipulate a future agent: `Skill("PromptInjection")` or
   `Skill("SecurityTriage")` on the flagged/whole content. Deterministic + adversarial = the
   safety-critical-content-gate doctrine.
6. **APPROVE + promote** — `bun PAI/Tools/okf-promote.ts <name> --by <who> --approve`. Gates:
   - **G1** independent HIGH re-scan → **0 HIGH** (fail-closed)
   - **G2** size/count caps (≤150 files, ≤5 MB; `--max-files`/`--files` to scope) → blocks mega-dumps
   - **G3** explicit `--approve` (human)
   Promotes sanitized copies into the **reference** tier, preserves/rewrites provenance frontmatter
   (`source: external`, `tier: reference`, `origin`, `promoted`, `promoted_by`, gates), appends
   `~/MEMORY/reference/okf/_ledger.jsonl`.
7. **Reversible** — un-promote = delete `~/MEMORY/reference/okf/<b>/` + its ledger line.

## Rules

- Default is **DRY RUN**. Promotion is opt-in per bundle, never bulk.
- **Fail-closed:** any HIGH finding or cap violation blocks promotion.
- **Provenance is mandatory** — recall must treat reference-tier hits as external-derived, never as
  Duane's own fact. (Recall wiring for the reference tier is a follow-up; the tag + namespace are
  the mechanism.)
- Additive-only: promotion never overwrites `~/MEMORY/auto/`.

## Tools
- `PAI/Tools/okf-discover.ts` — discovery sweep (+ `okf-discover.timer` weekly)
- `PAI/Tools/okf-import.ts` — clone-scan into quarantine
- `PAI/Tools/okf-promote.ts` — gated quarantine → reference promotion
- `PAI/Tools/crawl4ai.ts` — sovereign stealth crawler for bot-blocked sources (localhost, auth, injection-scanned output)
