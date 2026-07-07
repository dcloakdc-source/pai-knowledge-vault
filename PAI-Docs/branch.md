# Agent Memory — Branch Context

**Branch:** master (per CLAUDE.md operational rule: `~/.claude` repo always commits directly to master, no feature branches)
**Last updated:** 2026-05-18T22:15Z by session `family-touchscreen-agenda`

## Active Workstream
Family touchscreen agenda prototype — Tier A consensus design demonstrating the council's most-confident principles.

**Files:**
- `PAI/MEMORY/WORK/20260518-195900_family-agenda-prototype/family-agenda.html` — the prototype
- `PAI/MEMORY/WORK/20260518-195900_family-agenda-prototype/ISA.md` — Tier A design, 22/22 ISCs
- `PAI/MEMORY/WORK/20260518-205200_family-agenda-drag-drop/ISA.md` — drag-drop, 22/22 ISCs

## Goal
Demonstrate a household-information-appliance UX as a single self-contained HTML file. Validate the council's Tier A consensus by making it tactile (drag a child's task onto a parent's avatar; watch colors swap; receiving avatar pulses).

## Constraints
- Single self-contained HTML — no build tools, no dependencies, opens in any browser at 1280×800.
- Touchscreen primary — Pointer Events, never HTML5 native DnD.
- No login, no profile-switching — single shared surface, personalization is highlight-not-mode.
- `~/.claude` is PRIVATE forever — this prototype's design ideas can ship publicly, but the WORK ISA path itself must not appear in public docs.

## In Progress / Open
- **Interactive QR-handoff** — Tier B principle ("physical handoff to phone for complex edits"). Asked by user 2026-05-18 22:14Z; answered with recommendation (long-press → fullscreen QR encoding item state in URL); not yet implemented. User next step would be "build it" to proceed.
- **Ambient idle mode** — Tier C principle; no field data; not in prototype.
- **Real touchscreen validation** — current verification is headless-chromium simulated PointerEvents; the council's Tier A claims have not been tested on an actual kitchen tablet with a real family.

## Resume Prompt
> "Continue the family-touchscreen-agenda work. Latest state: drag-drop verified at `MEMORY/WORK/20260518-205200_family-agenda-drag-drop/`. Open ask: should I prototype the QR-handoff interaction (long-press → fullscreen QR card)? See `commit.md` 2026-05-18 entry for full context."

## Recent Pre-Session Context (carryover)
- Previous active workstream: skill curation (2026-05-16, see prior commit.md entry).
- Algorithm version: v5.7.2.
- PAI repo at master with ~80 deleted ISA files in `git status` from prior cleanup — unrelated to this session, do not stage with the agenda work.
