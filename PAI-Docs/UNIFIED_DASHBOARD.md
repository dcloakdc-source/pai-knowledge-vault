# Unified Dashboard Design
## Life + PAI + Synthesis in Single View

**Version:** 1.0  
**Created:** 2026-06-21  
**Purpose:** Single-pane-of-glass view across life governance, PAI governance, and synthesis layer  
**Command:** `/dashboard` (to be implemented as hook)

---

## Design Philosophy

**Principle:** The dashboard shows **what matters**, not **what's measurable**.

**Layout:** Three horizontal panels (Life, PAI, Synthesis) + Actions footer

**Refresh:** On-demand (`/dashboard`) + optional auto-refresh hooks (e.g., SessionStart)

**Data Sources:**
- Life: OKRS.md, health-metrics.jsonl, FAMILY.md
- PAI: ITIL metrics, ISA counts, practice maturity
- Synthesis: Alignment tracker, correlations, ROI

---

## Dashboard Layout

```
╔════════════════════════════════════════════════════════════════════╗
║                     UNIFIED DASHBOARD                               ║
║                     Duane's Life + PAI                             ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  🌟 LIFE (Missions & Health)                                       ║
║  ├─ M1: Optimize Health ━━━━━━━━━━░░░░░░░░░░ 45% (3/9 KRs on track) ║
║  ├─ M2: Revolutionize Learning ━━━━░░░░░░░░░░░░░░ 22% (2/9 KRs)    ║
║  ├─ M3: Inspire Collective Action ━░░░░░░░░░░░░░░░░░ 11% (1/9 KRs) ║
║  │                                                                 ║
║  ├─ Health: Sleep 6.2hr avg (⚠️ below 7hr target)                  ║
║  ├─ Energy: 5.8/10 avg this week (trending ↓)                     ║
║  └─ Family: 2/3 kids quality time this week ✓                     ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  🔧 PAI (Infrastructure & Governance)                              ║
║  ├─ Active ISAs: 3 (1 @ build, 2 @ verify)                        ║
║  ├─ Today's Sessions: 2 (1.2hrs, 145K tokens)                     ║
║  ├─ ITIL Maturity: 3.3/5.0 avg (6 @ 4/5, 14 @ 3/5, 0 below 3)     ║
║  │                                                                 ║
║  ├─ Security: ✅ No open problems (last scan: 2hr ago)             ║
║  ├─ Problems (7d): 0 critical, 1 warning, 3 info                  ║
║  └─ Services: 46/46 active ✓                                      ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  🔗 SYNTHESIS (Integration & Alignment)                            ║
║  ├─ Alignment Score: 67% (4/6 sessions trace to missions)         ║
║  ├─ Mission Distribution: M1 45%, M2 30%, M3 10%, unaligned 15%   ║
║  │                                                                 ║
║  ├─ Correlations Detected:                                        ║
║  │  • Sleep <7hr → Energy -1.2pts (p<0.05) ⚠️                     ║
║  │  • Exercise 4x/wk → Sleep +0.8hr (p<0.01) ✓                    ║
║  │                                                                 ║
║  ├─ PAI ROI: $X income enabled, $Y API cost = Z% ROI              ║
║  └─ Recommendations: [See below]                                  ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║  📋 QUICK ACTIONS                                                  ║
║  • /health-log     Log today's health metrics                     ║
║  • /okr-progress   Update OKR progress                            ║
║  • /itil           Show ITIL practice maturity                    ║
║  • /alignment      Show session → mission traces                  ║
╚════════════════════════════════════════════════════════════════════╝
```

---

## Panel 1: LIFE (Missions & Health)

### Mission Progress (TELOS OKRs)

**Data Source:** `PAI/USER/LIFE/OKRS.md` + `okr-progress.jsonl`

**Display:**
```
M1: Optimize Health ━━━━━━━━━━░░░░░░░░░░ 45% (3/9 KRs on track)
  ✓ KR1.1: Sleep ≥7hr (70/90 nights) → 32/90 so far ✓ on track
  ⚠ KR1.2: Exercise 4x/wk → 6/13 weeks ⚠ behind
  ✓ KR1.3: Energy avg ≥6/10 → 5.8/10 ⚠ slightly below
```

**Calculation:**
- Progress % = (days elapsed / days in quarter) vs (KR completion %)
- On track if actual ≥ projected
- Warning if actual < projected - 10%
- Critical if actual < projected - 25%

**Color Coding:**
- ✓ Green: On track or ahead
- ⚠ Yellow: Behind but recoverable
- ✗ Red: Significantly behind

---

### Health Metrics (M1 Focus)

**Data Source:** `health-metrics.jsonl`

**Display:**
```
Health Snapshot (Last 7 Days):
  Sleep:     6.2hr avg (⚠️ below 7hr target, trending ↓)
  Exercise:  2/7 days (⚠️ below 4x/week target)
  Energy:    5.8/10 avg (trending ↓ from 6.5 last week)
  Nutrition: 4/7 healthy meals ✓
```

**Alerts:**
- Sleep <6hr for 3+ consecutive days → ⚠️ recovery needed
- Exercise 0 days this week → ⚠️ movement reminder
- Energy <5/10 for 5+ days → ⚠️ burnout risk

---

### Family Metrics

**Data Source:** `FAMILY.md`, manual tracking

**Display:**
```
Family Quality Time (This Week):
  Child 1: ✓ 1.5hr one-on-one
  Child 2: ✓ 2hr one-on-one
  Child 3: ✗ 0hr (need to schedule!)
  
  Family Time: ✓ 3 shared meals, 1 outing
```

**Target:** ≥1hr one-on-one per child per week

---

## Panel 2: PAI (Infrastructure & Governance)

### Active Work (ISAs)

**Data Source:** `MEMORY/WORK/*/ISA.md` frontmatter

**Display:**
```
Active ISAs: 3
  • governance-frameworks (build) - 65/80 ISCs ━━━━━━━░░░
  • itil-session (verify) - 118/118 ISCs ━━━━━━━━━━ ✓
  • synthesis (plan) - 0/20 ISCs ░░░░░░░░░░░░░░░░░░░
```

**Sorting:** By phase (build first, then verify, etc.)

---

### Session Activity (Today)

**Data Source:** Session logs, work-log.jsonl

**Display:**
```
Today's Sessions: 2 (1.2hrs total, 145K tokens used)
  1. ITIL implementation (E5, 5.5hr, 167K tokens) ✓ complete
  2. Governance frameworks (E5, 1.2hr so far, 145K tokens) ⏳ active
```

---

### ITIL Maturity Overview

**Data Source:** `practice-maturity.json`

**Display:**
```
ITIL Maturity: 3.3/5.0 avg
  • 4/5: 6 practices (Problem Detection, RCA, Learning, etc.)
  • 3/5: 14 practices (Security Gov, Risk Mgmt, etc.)
  • 2/5: 0 practices ✓ (all above 2/5!)
  
Top 3 Improvement Opportunities:
  1. Change Control (3/5) → Target 4/5
  2. Capability Registry (3/5) → Target 4/5
  3. State Tracking (3/5) → Target 4/5
```

---

### Security & Operations

**Data Source:** `problem-metrics.jsonl`, systemd status

**Display:**
```
Security: ✅ No open problems (last scan: 2hr ago)
Problems (7d): 0 critical, 1 warning, 3 info

Services: 46/46 active ✓
  • Qdrant: ✓ healthy
  • Ollama: ✓ healthy
  • OmniPulse: ✓ healthy
```

---

## Panel 3: SYNTHESIS (Integration & Alignment)

### Alignment Score

**Data Source:** AlignmentTracker.ts

**Display:**
```
Alignment Score: 67% (4/6 sessions trace to missions)

Mission Distribution (Last 30 Days):
  M1 (Health):    ━━━━━━━━━░░░ 45% (27hr)
  M2 (Learning):  ━━━━━░░░░░░░ 30% (18hr)
  M3 (Resources): ━░░░░░░░░░░░ 10% (6hr)
  Unaligned:      ━━░░░░░░░░░░ 15% (9hr) ⚠️

Unaligned Work:
  • "Fix voice overlap bug" (1.5hr) - infrastructure, not mission
  • "Optimize dashboard" (2hr) - nice-to-have
```

**Goal:** ≥80% aligned, ≤20% unaligned

---

### Correlations Detected

**Data Source:** Correlation engine (health × productivity × mission progress)

**Display:**
```
Significant Correlations:
  • Sleep <7hr → Energy -1.2pts (p<0.05) ⚠️
    Action: Protect sleep to maintain energy
    
  • Exercise 4x/wk → Sleep +0.8hr (p<0.01) ✓
    Action: Keep exercising (it helps sleep!)
    
  • Energy ≥7/10 → ISC pass rate +12% (p<0.05) ✓
    Action: Health directly impacts work quality
```

**Method:** Pearson correlation on 30+ data points, p-value <0.05 threshold

---

### PAI ROI Calculation

**Data Source:** API cost tracking, income attribution, time saved

**Display:**
```
PAI ROI (Q3 2026):
  Income Enabled:  $X (consulting, teaching enabled by PAI)
  Time Saved:      Yhr (automation, faster delivery)
  Operating Cost:  $Z (API costs, hardware amortization)
  
  ROI: [(X + (Y × hourly_rate)) - Z] / Z = ABC%
  
  Verdict: ✓ PAI investment paying off
```

**Future:** Track quarterly, adjust investment based on ROI

---

### Recommendations (AI-Generated)

**Data Source:** Pattern analysis across all three panels

**Display:**
```
🎯 Top Recommendations:
  1. Priority: Increase sleep (below target, affects energy)
  2. Schedule: Child 3 quality time (missing this week)
  3. Focus: Continue M1 work (highest alignment, measurable progress)
  4. Optimize: Reduce unaligned PAI work (15% not mission-related)
```

**Criteria:**
- Data-driven (not subjective)
- Actionable (specific next step)
- Prioritized (impact × effort)

---

## Implementation Approach

### Phase 1: Hook-Based Command (This Session)
**File:** `/home/duane/.claude/hooks/UnifiedDashboard.hook.ts`  
**Trigger:** User types `/dashboard`  
**Action:** Read data sources, format output, display

**Data Sources:**
- OKRS.md (parse markdown)
- health-metrics.jsonl (last 7 days)
- practice-maturity.json (latest)
- problem-metrics.jsonl (last 7 days)
- work-log.jsonl (session activity)
- alignment-scores.jsonl (AlignmentTracker output)

**Output:** Text-based dashboard (as shown above)

---

### Phase 2: Auto-Refresh (Future)
**Trigger:** SessionStart hook  
**Action:** Show dashboard at session start (optional, configurable)

**Use Case:** Every morning, see unified view before deciding what to work on

---

### Phase 3: Dashboard Enhancements (Future)
**Visual:** HTML dashboard served via PAI Command Center (:8766)  
**Interactive:** Click KRs to see details, charts over time  
**Mobile:** Dashboard accessible from phone (family coordination)

---

## Quick Actions

### /health-log
**Purpose:** Log today's health metrics  
**Prompts:**
- Sleep hours (e.g., 7.5)
- Exercise? (yes/no)
- Energy level (1-10)
- Healthy meals count (0-3)

**Output:** Appends to `health-metrics.jsonl`, updates dashboard

---

### /okr-progress
**Purpose:** Update OKR progress  
**Prompts:**
- Which KR? (select from list)
- Current value? (e.g., 32 nights of ≥7hr sleep)

**Output:** Updates `okr-progress.jsonl`, recalculates %

---

### /alignment
**Purpose:** Show session → mission traces  
**Output:**
```
Session Alignment Report:
  ✓ governance-frameworks → M1 (health foundation enables work)
  ✓ itil-implementation → M1 + M2 (learning system)
  ⚠ voice-overlap-bug → Unaligned (infrastructure polish)
```

---

## Success Metrics

**Dashboard is successful if:**
1. Duane checks it weekly (usage tracking)
2. Alignment score improves over time (trend)
3. Correlations actionable (acted upon ≥50%)
4. Saves time vs. manual status checks (≥15min/week)
5. Influences decisions ("saw dashboard, chose to sleep instead of code")

---

## Privacy & Boundaries

**What Dashboard Shows:**
- Duane's metrics (sleep, energy, OKRs)
- PAI system status (public knowledge)
- Family time (aggregate only, not specifics)

**What Dashboard Does NOT Show:**
- Kids' individual data (unless they consent)
- Financial details (keep those in YNAB/private)
- Sensitive health info (mental health details private)

**Access Control:**
- Dashboard = Duane only (personal data)
- Kids could have separate family-focused view (if wanted)

---

**Next Steps:**
1. Build UnifiedDashboard.hook.ts (this session)
2. Test with current data (OKRS.md, ITIL metrics)
3. Iterate based on usage (add/remove panels)
4. Add visual version (future)

---

**Created:** 2026-06-21  
**Status:** Design complete, implementation next  
**Version:** 1.0
