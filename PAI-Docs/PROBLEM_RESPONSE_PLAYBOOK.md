# Problem Response Playbook

**ITIL Practice:** Problem Detection & Response  
**Version:** 1.0  
**Last Updated:** 2026-06-21

---

## Overview

This playbook defines standard response procedures for problems detected by PAI's automated problem detection system. Problems are logged to `PAI/MEMORY/STATE.claude/problem-metrics.jsonl` by security hooks and monitoring tools.

---

## Severity Matrix

| Severity | Definition | Response Time | Escalation | Example |
|----------|------------|---------------|------------|---------|
| **CRITICAL** | System security breach, data loss risk, or complete functionality failure | Immediate | Principal notification | DestructiveOpGuard blocks `rm -rf /` |
| **WARNING** | Potential issue, degraded functionality, or policy violation | Within 24 hours | Log review | IdentityValidator detects fabrication |
| **INFO** | Informational event, no action required | Log only | None | Routine security scan |

---

## Response Workflows

### 1. Triage (All Severities)

**Objective:** Classify the problem and determine response priority.

**Steps:**
1. Review problem event in `problem-metrics.jsonl`
2. Check if problem is recurring (≥3 occurrences same hook)
3. Assess impact:
   - Critical: Immediate security/data risk
   - Warning: Degraded functionality or policy concern
   - Info: Informational only
4. Route to appropriate workflow below

**Tools:**
```bash
# View recent problems
bun ~/.claude/PAI/Tools/ProblemMetrics.ts --report --days 7

# Check for patterns
bun ~/.claude/PAI/Tools/ProblemMetrics.ts --stats
```

---

### 2. Investigate (Critical & Warning)

**Objective:** Understand root cause and gather evidence.

**Steps:**
1. **Reproduce** (if possible):
   - What command/action triggered the block?
   - Can it be reproduced consistently?

2. **Gather Context**:
   - Check security event logs: `MEMORY/SECURITY/YYYY/MM/security-*.jsonl`
   - Review session context if `session_id` available
   - Check ISA/PRD if `prd_slug` present

3. **Assess Intent**:
   - Was this a legitimate operation blocked by overly strict rules?
   - Was this a genuine security threat?
   - Was this operator error?

4. **Determine Root Cause**:
   - **For recurring problems (≥5 occurrences)**, RCA is REQUIRED:
     ```bash
     # Check for patterns first
     bun ~/.claude/PAI/Tools/ProblemMetrics.ts --patterns
     
     # Create postmortem
     bun ~/.claude/PAI/Tools/CreatePostmortem.ts "Pattern: HookName:severity"
     
     # Run RCA in Claude Code
     @RootCauseAnalysis Please analyze this recurring problem: [describe pattern]
     ```
   
   - **For one-time critical incidents**, RCA is RECOMMENDED:
     ```bash
     # Create postmortem for major incidents
     bun ~/.claude/PAI/Tools/CreatePostmortem.ts "Incident Title" --severity Critical
     ```

**Evidence Checklist:**
- [ ] Problem event JSON
- [ ] Security log entry
- [ ] Session context
- [ ] Reproduction steps
- [ ] Root cause hypothesis
- [ ] RCA output (if recurring ≥5 times)

---

### 3. Resolve (Critical & Warning)

**Objective:** Fix the underlying issue and prevent recurrence.

#### Resolution Paths

**Path A: Legitimate Operation (False Positive)**

If investigation confirms the blocked operation was legitimate:

1. **Grant Override** (one-time):
   - Tell Claude: "override once"
   - Operation proceeds, but rule remains active

2. **Update Security Rules** (recurring false positives):
   - Edit `~/MEMORY/STATE.claude/destructive-patterns.json` (for DestructiveOpGuard)
   - OR edit `~/.claude/hooks/patterns.yaml` (for SecurityValidator)
   - Mark pattern as `warn_only: true` or remove entirely
   - Document rationale in git commit

3. **Session Override** (legitimate for this session):
   - Tell Claude: "override session"
   - Rule bypassed for current session only

4. **Permanent Override** (always safe):
   - Tell Claude: "override permanent"
   - Add to `settings.json → permissions.allow`

**Path B: Genuine Security Threat**

If investigation confirms a real threat:

1. **Block Confirmed** - no override
2. **Investigate Attack Vector**:
   - How did this command get triggered?
   - Was it AI hallucination?
   - Was it user error?
   - Was it adversarial prompt injection?

3. **Strengthen Defenses**:
   - Add more specific patterns if needed
   - Update prompts/instructions to avoid this class of error
   - Consider additional hooks/guardrails

**Path C: Configuration Error**

If the hook itself is misconfigured:

1. **Fix Hook Logic**:
   - Edit hook file
   - Test with known-good and known-bad inputs
   - Commit fix

2. **Regression Test**:
   - Ensure fix doesn't break existing protections
   - Run hook test suite if available

---

### 4. Learn (All Severities)

**Objective:** Capture lessons and update system knowledge.

**Steps:**
1. **Create Feedback Memory** (for failures/false positives):
   ```bash
   # Create feedback memory documenting the issue
   touch ~/.claude/PAI/MEMORY/Feedback/feedback_[describe-issue].md
   ```

   Template:
   ```markdown
   # Feedback: [Issue Description]

   **Date:** YYYY-MM-DD  
   **Problem:** [What happened]  
   **Root Cause:** [Why it happened]  
   **Resolution:** [How it was fixed]  
   **Lesson:** [What we learned]  
   **Prevention:** [How to avoid recurrence]
   ```

2. **Update Documentation** (if process gap found):
   - Update this playbook if response wasn't clear
   - Update hook documentation if behavior was unexpected

3. **Close Problem**:
   - No formal closure in current system
   - Problem remains in log for metrics/pattern analysis
   - Future: Add `resolution` field to problem events

---

## Common Problems & Solutions

### DestructiveOpGuard Blocks

| Problem | Root Cause | Solution |
|---------|------------|----------|
| Blocked: `rm -rf /tmp/build` | Overly broad `rm -rf` pattern | Add `/tmp/` to safe paths OR use `rm -rf /tmp/build/*` instead |
| Blocked: `dd if=backup.img of=/dev/sdb` | Legitimate disk restore | Override once, verify target device first |
| Blocked: `smbclient ... del *` | Recursive delete in SMB share | Verify share path, override if correct |

### SecurityValidator Blocks

| Problem | Root Cause | Solution |
|---------|------------|----------|
| Blocked: Edit outside workspace | ContainmentGuard path traversal protection | Add path to containment zones OR work inside workspace |
| Blocked: Read `/etc/passwd` | Sensitive file protection | Override if legitimate system admin task |
| Blocked: Write to `/usr/local/bin` | System directory protection | Use `~/.local/bin` instead OR override if installing global tool |

### IdentityValidator Warnings

| Problem | Root Cause | Solution |
|---------|------------|----------|
| Warning: Fabricated file path | AI hallucination of non-existent file | Use Read tool to verify files exist before claiming presence |
| Warning: Unsourced claim | AI assertion without evidence | Always verify with tools before asserting state |

### PhaseTransitionGate Warnings

| Problem | Root Cause | Solution |
|---------|------------|----------|
| Warning: PLAN→BUILD without approval | Skipped planning phase | Complete ISC in OBSERVE/PLAN before BUILD |
| Warning: Premature phase transition | Missing verification | Complete current phase ISCs before advancing |

---

## Integration Points

### RootCauseAnalysis Skill

**When to Use RCA:**
- ✅ **REQUIRED:** Recurring problems ≥5 occurrences
- ✅ **RECOMMENDED:** Critical one-time incidents
- ✅ **OPTIONAL:** Major incidents with unclear cause

**RCA Workflow:**

1. **Detect Pattern**
   ```bash
   bun ~/.claude/PAI/Tools/ProblemMetrics.ts --patterns
   ```

2. **Create Postmortem**
   ```bash
   # For recurring pattern
   bun ~/.claude/PAI/Tools/CreatePostmortem.ts "Pattern: HookName:severity" --pattern "HookName:severity"
   
   # For one-time incident
   bun ~/.claude/PAI/Tools/CreatePostmortem.ts "Incident Title" --severity Critical
   ```

3. **Run RCA in Claude Code**
   ```
   @RootCauseAnalysis

   Problem: [Hook name] has blocked [operation] 5 times in the last week.

   Context:
   - Hook: [hook name]
   - Pattern: [common pattern across occurrences]
   - Sessions: [affected session IDs if available]

   Please perform a root cause analysis using [Five Whys | Fishbone | Fault Tree].
   ```

4. **Document in Postmortem**
   - Paste full RCA output into postmortem
   - Extract root causes
   - Define prevention actions

5. **Create Feedback Memories**
   ```bash
   touch ~/.claude/PAI/MEMORY/Feedback/feedback_[lesson-slug].md
   ```
   Fill in template (see POSTMORTEM_TEMPLATE.md)

6. **Track Actions**
   - Add prevention items to postmortem
   - Assign owners and ETAs
   - Set follow-up review date

**RCA Methods Available:**
- **Five Whys:** Linear cause chains, simple problems
- **Fishbone (Ishikawa):** Complex multi-factor problems
- **Fault Tree:** Safety-critical, AND/OR logic gates
- **Kepner-Tregoe:** Subtle/hard-to-reproduce defects
- **Postmortem:** Incident timeline + blameless culture

**Files:**
- **Template:** `PAI/DOCUMENTATION/POSTMORTEM_TEMPLATE.md`
- **Storage:** `PAI/MEMORY/RCA/YYYY-MM-DD_incident-slug.md`
- **Tool:** `PAI/Tools/CreatePostmortem.ts`

### Monthly Review Integration

Problem metrics feed into the monthly ITIL review:
- Recurring problem count
- Top problem-generating hooks
- Resolution time (when tracking added)
- Pattern trends

Recommendations generated automatically by `ITILMonthlyReview.ts`.

---

## Quick Reference

### Check Problem Status
```bash
bun ~/.claude/PAI/Tools/ProblemMetrics.ts --stats
```

### View Recent Problems
```bash
bun ~/.claude/PAI/Tools/ProblemMetrics.ts --report --days 7
```

### Grant Override
In Claude Code, say:
- `"override once"` - This operation only
- `"override session"` - This session only  
- `"override permanent"` - Always allow

### Create Feedback Memory
```bash
touch ~/.claude/PAI/MEMORY/Feedback/feedback_[issue-slug].md
```

### Root Cause Analysis
```
@RootCauseAnalysis [describe recurring problem]
```

---

## Metrics & Continuous Improvement

### Tracked Metrics
- Total problems detected (by severity)
- Problems by hook
- Blocked vs. overridden ratio
- Recurring problem patterns
- (Future) Mean time to resolution

### Review Cadence
- **Weekly:** Check problem stats via `/itil metrics`
- **Monthly:** Full review via `ITILMonthlyReview.ts`
- **Quarterly:** Pattern analysis and hook effectiveness review

### Improvement Triggers
- **≥10 blocks/week same pattern** → Review rule specificity
- **Override rate >30%** → Rules too strict, need refinement
- **Recurring problem ≥5 times** → Mandatory RCA
- **Zero problems for 30 days** → Verify detection is working

---

## Escalation

### When to Escalate to Principal

1. **Critical security breach detected** - Immediate notification
2. **Recurring problem with no clear resolution** - Weekly notification
3. **Hook malfunction (fail-open)** - Immediate notification
4. **False positive rate >50%** - Monthly review discussion

### Escalation Format

```markdown
Subject: [CRITICAL/WARNING] Problem Detection Alert

Severity: [Critical/Warning]
Hook: [Hook name]
Occurrences: [Count in last 7 days]
Pattern: [Common pattern]

Summary: [2-3 sentence description]

Action Required: [What principal needs to decide]

Context: [Link to problem metrics or logs]
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-06-21 | Initial playbook (Problem Detection practice 0→3) |

