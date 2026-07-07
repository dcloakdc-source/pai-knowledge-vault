---
title: "MultiEngineSynthesis — Cross-Engine Consolidation Methodology"
type: idea
tags: [methodology, synthesis, assessment, multi-engine, skill]
created: 2026-06-19
updated: 2026-06-19
quality: 5
source_session: 20260619-120000_consolidate-altana-cross-engine
related:
  - slug: secret-pattern-consolidation
    type: related
---

# MultiEngineSynthesis

Formal methodology for consolidating independent assessments, reviews, or research from multiple PAI engines (PNC, PNK, PNX, Gemini) into a single authoritative output.

## Thesis

Four independent AI engines assessing the same target produce complementary blind spots, different frameworks, and orthogonal coverage. The methodology captures a repeatable pattern for reconciling these: DISCOVER → LOAD → COMPARE → RESOLVE → CONSOLIDATE → VERIFY → APPLY. Each engine's unique findings are preserved — consolidation is not flattening.

## Evidence

Proven on the Altana Product Passport assessment: 4 engines, 10 resolved contentions (K1-K10), 513-line consolidated output with 20 risks, 9 sections. Each engine contributed unique findings that would have been missed in a single-engine assessment.

## Key Design Decisions

- **Canonical spine selection:** The engine with the broadest framework coverage becomes the backbone. STRIDE (PNC) subsumed custom categories (PNK).
- **Precision overlay:** Qualitative scoring primary (more honest), numeric overlaid for cross-referencing.
- **Actor-actionable routing:** Vendor-side capabilities become DDQ items, not customer mitigations.
- **6 resolution rules** in priority order: superset > precision > specialization > actor-actionable > conservative > escalate.

## Implications

- Multi-engine consolidation is not optional for thorough assessments — no single engine covers all dimensions
- Skills directory at `~/.opencode/skills/MultiEngineSynthesis/` (PNK tree, 8 files, ~1133 lines)
- Algorithm integration documented at AlgorithmPattern.md as `mode: synthesis` sub-phase (4b/7)
- Applicable to security assessments, code reviews, and research synthesis
