# ITIL Operational Implementation Guide
## How to Integrate Framework into Daily PAI Operations

**Version:** 1.0  
**Created:** 2026-06-21  
**Purpose:** Step-by-step guide for operationalizing ITIL framework

---

## Quick Start

### 1. Run Initial Metrics Collection
```bash
cd ~/.claude && bun PAI/Tools/ITILMetricsCollector.ts
```

This establishes your baseline across all 6 KPIs.

### 2. Assess Practice Maturity
```bash
bun PAI/Tools/PracticeMaturity.ts --weakest 5
```

Identifies your 5 weakest practices for focused improvement.

### 3. Review Service Catalog
```bash
cat PAI/DOCUMENTATION/SERVICE_CATALOG.md
```

Understand what capabilities are available.

### 4. Set Up Monthly Review
Add to crontab or systemd timer:
```bash
# Run monthly review on 1st of each month
0 9 1 * * cd ~/.claude && bun PAI/Tools/ITILMonthlyReview.ts
```

---

## Daily Operations

### Before Starting Work

**1. Check applicable guiding principle:**

| Task Type | Apply Principle |
|-----------|-----------------|
| Bug fix / debugging | **Start Where You Are** - Reproduce before reading code |
| New feature | **Focus on Value** - Does this create principal value? |
| Refactoring | **Keep It Simple** - Remove complexity, don't add |
| Multi-step work | **Progress Iteratively** - Break into phases, verify each |
| Cross-engine work | **Collaborate & Visibility** - Use shared ICM memory |
| Architecture decision | **Think Systemically** - What else does this affect? |
| Proven workflow | **Optimize & Automate** - Can this be a hook/skill? |

**2. Run existence probe (Preflight Gate E):**
```bash
# Before building anything new
rg "SEARCH_TERM" ~/.claude/PAI/ --type md -l
```

Ensures you're not duplicating existing work.

### During Work

**Algorithm OBSERVE Phase:**
- Voice announcement: "Entering Algorithm"
- Auto-print applicable guiding principle (TODO: integrate into template)
- Run preflight gates
- Select capabilities

**Algorithm REFLECT Phase:**
- Auto-check: Did failed ISCs generate feedback memories?
- If no: Warning + prompt to create memory before phase=complete

### After Work

**Run Simplify:**
```bash
# Post-implementation cleanup
bun PAI/Tools/Simplify.ts
```

**Update Metrics:**
```bash
# Refresh ITIL dashboard
bun PAI/Tools/ITILMetricsCollector.ts
```

---

## Weekly Operations

### Monday: Review Dashboard

Check ITIL CSF/KPI panel (once integrated):
- Are any KPIs red (fail)?
- What's the trend (improving/declining)?
- Do any metrics need investigation?

### Friday: Practice Check

Quick self-assessment:
- Which ITIL practice did I use most this week?
- Which practice did I skip that I should have used?
- Did I follow the seven guiding principles?

---

## Monthly Operations

### First Week of Month

**1. Run Full Assessment:**
```bash
cd ~/.claude
bun PAI/Tools/ITILMetricsCollector.ts
bun PAI/Tools/PracticeMaturity.ts
bun PAI/Tools/ITILMonthlyReview.ts
```

**2. Review Monthly Report:**
Located at `MEMORY/WORK/YYYYMM-01-000000_itil-monthly-review/REPORT.md`

**3. Implement Top Recommendation:**
- Pick the highest-priority critical recommendation
- Create work item in TELOS or GitHub
- Schedule implementation this month

**4. Update Service Catalog:**
```bash
bun PAI/Tools/GenerateServiceCatalog.ts
```

Ensures new skills/agents are documented.

---

## Quarterly Operations

### Review & Refine

**1. TELOS Context Refresh:**
Use Interview skill to update principal context.

**2. Practice Maturity Trends:**
Compare current maturity scores to 3 months ago:
```bash
# Review historical practice-maturity.json versions
git log -p -- MEMORY/STATE.claude/practice-maturity.json
```

**3. Framework Alignment Check:**
- Are the seven principles still relevant?
- Do KPI targets need adjustment?
- Should any practices be added/removed?

**4. Improvement Review:**
- Which recommendations were implemented?
- What was the measurable impact?
- Which gaps remain?

---

## Integration Points

### Algorithm Template Updates

**OBSERVE Phase (After Voice):**
```markdown
**📋 APPLICABLE GUIDING PRINCIPLE:**

[Auto-detect from task type and print relevant principle]

Examples:
- Bug fix → "Start Where You Are: Reproduce failure before code analysis"
- New feature → "Focus on Value: Every action must create stakeholder value"
- Refactor → "Keep It Simple: Use minimum steps to achieve objective"
```

**REFLECT Phase (Before phase=complete):**
```markdown
**✅ LEARNING CAPTURE CHECK:**

Failed ISCs: ${failedISCCount}
Feedback Memories Created: ${feedbackMemoryCount}

[If feedbackMemoryCount < failedISCCount]
⚠️  WARNING: ${failedISCCount - feedbackMemoryCount} failed ISCs lack feedback memories.
Create feedback memories before marking phase=complete.
```

### Hook Integration

**New Hook: `ITILPrincipleReminder.hook.ts`**
- **Trigger:** SessionStart
- **Action:** Print applicable guiding principle based on detected task type
- **Implementation:** Maps task keywords to principles

**Enhanced Hook: `PhaseTransitionGate.hook.ts`**
- **Addition:** Check feedback memory creation on REFLECT→COMPLETE transition
- **Action:** Block transition if failed ISCs have no memories (warn + bypass option)

### Dashboard Panel

**ITIL CSF/KPI Panel Structure:**
```
┌─ ITIL Service Management ──────────────────┐
│                                             │
│ Value Delivery:    8.5/10  ✅  [▃▅▆▇█]     │
│ Quality:           92%     ⚠️   [▃▄▄▅▆]     │
│ Learning Capture:  0.8     ❌  [▁▂▂▃▃]     │
│ Efficiency:        45%     ❌  [▁▁▂▂▃]     │
│ Reliability:       99.5%   ✅  [▆▇▇██]     │
│ Security:          100%    ✅  [█████]     │
│                                             │
│ [View Details] [Monthly Review]             │
└─────────────────────────────────────────────┘
```

Sparklines show 7-day trend, status symbols indicate vs. target.

---

## Measurement Details

### Data Collection Points

| Metric | Source | Collection Method |
|--------|--------|-------------------|
| ISC Pass Rate | ISA files | Parse `## Criteria` section, count `[x]` vs total |
| Euphoric Surprise | ISA frontmatter | Extract `euphoric_surprise:` field |
| Feedback Memories | MEMORY/Feedback/ | Count .md files |
| Ollama Routing | Inference.ts logs | Parse calls, calculate Ollama/(Ollama+Claude) % |
| Uptime | HealthCheck history | Track probe successes over time |
| Security Compliance | DestructiveOpGuard logs | Count blocks/(blocks+overrides) % |

### Storage Format

**Metrics:** `MEMORY/STATE.claude/itil-metrics.jsonl`
```json
{
  "timestamp": "2026-06-21T12:00:00Z",
  "valueDelivery": { "euphoricSurpriseAvg": 8.5, "target": 8, "status": "pass" },
  "quality": { "iscPassRate": 92, "target": 95, "status": "warn" },
  ...
}
```

**Practice Maturity:** `MEMORY/STATE.claude/practice-maturity.json`
```json
{
  "timestamp": "2026-06-21T12:00:00Z",
  "practices": [
    { "practice": "Problem Detection", "score": 3, "evidence": [...], "gaps": [...] }
  ],
  "avgScore": 2.8,
  "weakest": ["Practice A", "Practice B", ...]
}
```

---

## Troubleshooting

### "Metrics show 0% across the board"

**Cause:** No completed ISAs in MEMORY/WORK/ or paths incorrect.

**Fix:**
1. Check PAI_DIR environment variable: `echo $PAI_DIR`
2. Verify ISA files exist: `ls ~/.claude/PAI/MEMORY/WORK/*/ISA.md | head -5`
3. Re-run collector: `bun PAI/Tools/ITILMetricsCollector.ts`
4. Inspect recent hook logs if the collector still reports zero metrics.

### "Practice maturity scores too low"

**Cause:** Tool is checking ~/.claude paths, may be run from wrong environment.

**Fix:** Always run from PNC (Claude Code) environment where full PAI structure exists.

### "Monthly review generates no recommendations"

**Cause:** All CSFs passing or insufficient metrics history.

**Fix:**
1. Check metrics file: `wc -l MEMORY/STATE.claude/itil-metrics.jsonl`
2. Need at least 2 metric snapshots for trends
3. Run metrics collector multiple times over days

---

## Success Criteria

After full implementation, you should see:

✅ **Metrics collected automatically** (weekly cron job)  
✅ **Dashboard shows all 6 KPIs** with real-time status  
✅ **Failed ISCs auto-checked** for feedback memories  
✅ **Guiding principles surface** at decision points  
✅ **Monthly reviews generate** actionable recommendations  
✅ **Practice maturity tracked** with clear improvement path  
✅ **Service catalog up-to-date** with all capabilities  

---

## Next Steps

1. **Immediate:** Run initial baseline metrics
2. **This Week:** Integrate guiding principles into Algorithm template
3. **This Month:** Add ITIL panel to dashboard
4. **This Quarter:** Implement top 3 practice maturity improvements

---

**Related Files:**
- `ITIL_FRAMEWORK.md` - Complete framework documentation
- `ITIL_QUICK_REFERENCE.md` - Daily-use cheat sheet
- `SERVICE_CATALOG.md` - Capability registry
- `ITILMetricsCollector.ts` - Metrics collection tool
- `PracticeMaturity.ts` - Maturity assessment tool
- `ITILMonthlyReview.ts` - Review automation

**Questions?** See framework documentation or run tools with `--help` flag.
