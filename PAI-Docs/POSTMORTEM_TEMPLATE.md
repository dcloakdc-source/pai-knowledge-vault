# Postmortem: [Incident Title]

**Date:** YYYY-MM-DD  
**Author:** [Name or DA]  
**Incident Severity:** [Critical | Major | Minor]  
**Problem Pattern:** [Link to problem-metrics pattern if recurring]  
**RCA Method:** [Five Whys | Fishbone | Fault Tree | Kepner-Tregoe | Other]

---

## Executive Summary

[2-3 sentence summary of what happened, impact, and resolution]

**Impact:**
- Duration: [X hours/minutes]
- Affected Systems: [List]
- User Impact: [Description]
- Data Loss: [Yes/No - Details]

---

## Timeline (Blameless)

All times UTC. Focus on events, not individuals.

| Time | Event | Evidence |
|------|-------|----------|
| HH:MM | [What happened] | [Link to logs/screenshots/tool output] |
| HH:MM | [Detection] | [How we discovered the issue] |
| HH:MM | [Investigation started] | [Initial hypothesis] |
| HH:MM | [Root cause identified] | [Evidence] |
| HH:MM | [Fix applied] | [What was changed] |
| HH:MM | [Verification] | [How we confirmed resolution] |
| HH:MM | [Incident closed] | [Final status] |

---

## What Happened (Narrative)

[Detailed description of the incident from start to finish. Be specific about:
- Initial trigger/symptoms
- How the problem manifested
- How it was detected
- Investigation steps taken
- Resolution actions
- Verification that issue was resolved]

---

## Root Cause Analysis

### Method Used: [Five Whys | Fishbone | etc.]

[If using RootCauseAnalysis skill, paste the full output here]

### Primary Root Cause

[The fundamental cause that, if fixed, prevents recurrence]

**Evidence:**
- [Supporting data/logs]
- [Tool output]
- [Reproduction steps]

### Contributing Factors

[Secondary causes that made the problem worse or harder to detect]

1. **[Factor 1]**
   - How it contributed: [Explanation]
   - Evidence: [Links/data]

2. **[Factor 2]**
   - How it contributed: [Explanation]
   - Evidence: [Links/data]

### Why This Wasn't Caught Earlier

[Defense-in-depth analysis - which layers failed?]

- **Detection:** [Why didn't monitoring catch this?]
- **Prevention:** [Why didn't hooks/gates prevent this?]
- **Testing:** [Why didn't verification catch this?]

---

## Prevention & Remediation

### Immediate Actions (Already Taken)

- [x] [Action 1 - what was done immediately to stop the bleeding]
- [x] [Action 2 - temporary fix or workaround]

### Short-term Fixes (This Week)

Priority fixes to prevent immediate recurrence:

- [ ] **[Fix 1]**
  - Owner: [Name/Role]
  - ETA: [Date]
  - Verification: [How we'll confirm it works]

- [ ] **[Fix 2]**
  - Owner: [Name/Role]
  - ETA: [Date]
  - Verification: [How we'll confirm it works]

### Long-term Improvements (This Quarter)

Systemic changes to prevent entire class of problems:

- [ ] **[Improvement 1]**
  - Owner: [Name/Role]
  - ETA: [Date]
  - Success Metric: [How we'll measure effectiveness]

- [ ] **[Improvement 2]**
  - Owner: [Name/Role]
  - ETA: [Date]
  - Success Metric: [How we'll measure effectiveness]

---

## Lessons Learned

### What Went Well

[Things that worked during detection, investigation, or resolution]

1. [Positive observation 1]
2. [Positive observation 2]

### What Could Be Improved

[Gaps in process, tools, or knowledge - **not blaming people**]

1. [Gap 1]
   - Why it matters: [Impact]
   - Recommendation: [How to fix]

2. [Gap 2]
   - Why it matters: [Impact]
   - Recommendation: [How to fix]

### Knowledge Gaps Identified

[What we didn't know that we should document]

1. [Gap 1] → Create feedback memory: `feedback_[topic].md`
2. [Gap 2] → Update documentation: [File name]

---

## Feedback Memories Created

[List of feedback memories created from this postmortem]

- [ ] `MEMORY/Feedback/feedback_[topic-1].md` - [Brief description]
- [ ] `MEMORY/Feedback/feedback_[topic-2].md` - [Brief description]

**Template for feedback memory:**
```markdown
# Feedback: [Topic]

**Date:** YYYY-MM-DD
**Source:** Postmortem [link to this file]
**Root Cause:** [Primary root cause from RCA]

## What Happened
[Brief summary]

## Why It Happened
[Root cause in 1-2 sentences]

## How We Fixed It
[Resolution in 1-2 sentences]

## Lesson Learned
[The key takeaway - what to remember]

## Prevention
[How to avoid this in the future]

**Related:**
- Problem Pattern: [link if recurring]
- RCA Output: [link to RootCauseAnalysis skill output]
```

---

## Recurrence Prevention Checklist

Verify these actions to prevent recurrence:

### Detection
- [ ] Monitoring added for early warning signs
- [ ] Alerts configured with appropriate thresholds
- [ ] Dashboard shows relevant health metrics

### Prevention  
- [ ] Hooks/gates updated to catch this class of error
- [ ] Input validation strengthened
- [ ] Permissions/access controls tightened

### Testing
- [ ] Test case added to reproduce original failure
- [ ] Verification step added to relevant workflows
- [ ] Regression test prevents future breakage

### Documentation
- [ ] Runbook updated with troubleshooting steps
- [ ] Known issues documented
- [ ] Team knowledge shared (feedback memories)

---

## Related Links

- **Problem Metrics:** [Link to problem-metrics.jsonl entry or --patterns output]
- **RCA Output:** [Link to RootCauseAnalysis skill output if saved]
- **Security Logs:** [Link to relevant security events]
- **ISA/PRD:** [Link if incident occurred during specific project]
- **Git Commits:** [Links to fix commits]

---

## Sign-off

**Postmortem Reviewed By:**
- [Name/Role] - [Date]
- [Name/Role] - [Date]

**Action Items Tracked In:** [Link to tracking system or ISA]

**Follow-up Date:** [When to review if actions were effective]

---

## Appendices

### A. Full RCA Output

[If using RootCauseAnalysis skill, paste complete output here]

```
[RCA output from skill]
```

### B. Relevant Logs

[Key log excerpts - full logs should be linked, not pasted]

```
[Excerpt 1]
```

### C. Evidence Screenshots

[Links to screenshots if referenced in timeline]

---

**Template Version:** 1.0  
**Based On:** Google SRE Postmortem Culture, ITIL Problem Management
