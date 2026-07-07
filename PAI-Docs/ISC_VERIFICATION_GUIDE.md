# ISC Verification Guide

**Purpose:** Ensure all Ideal State Criteria (ISCs) have verification evidence before marking work complete.

**Target Audience:** Agents running the Algorithm VERIFY phase, engineers doing manual verification.

**Tool:** `PAI/Tools/ISCVerifier.ts`

---

## Quick Start

### During Algorithm VERIFY Phase

**After completing all work, before marking `phase: complete`:**

```bash
# Check current ISA
bun PAI/Tools/ISCVerifier.ts

# If any ISCs are missing, get a checklist template
bun PAI/Tools/ISCVerifier.ts --checklist
```

**Exit codes:**
- `0` - All ISCs verified (100%), safe to mark complete
- `1` - Some ISCs missing verification, **do not mark complete**
- `2` - Error (no ISA found, invalid format)

---

## What Is ISC Verification?

### The Problem

An ISC (Ideal State Criterion) defines what "done" means for a specific aspect of the work:

```markdown
## Criteria
- [ ] ISC-1: Web server responds with 200 status code
- [ ] ISC-2: Database contains at least 3 backup entries
- [ ] ISC-3: No plaintext secrets in committed files
```

**Verification evidence** proves each criterion is actually met:

```markdown
## Verification
- ISC-1: curl localhost:8080 returns 200 OK with expected HTML
- ISC-2: SELECT COUNT(*) FROM backups returns 5 rows
- ISC-3: git log -p | rg 'API_KEY|password' returns no matches
```

### Why It Matters

**Without verification:**
- Work appears "done" but may not actually meet criteria
- Technical debt accumulates (unverified features)
- Rework risk (discovering issues post-completion)
- Governance gap (can't prove completeness)

**With verification:**
- Concrete evidence that criteria are met
- Trustworthy "done" status
- Audit trail for compliance
- Confidence in quality

---

## Verification Standards

### What Counts as "Verification Evidence"

✅ **Good Evidence** (Concrete, Specific, Reproducible)

```markdown
- ISC-1: curl https://api.example.com/health returns {"status":"ok","version":"1.2.3"}
- ISC-2: ls ~/.claude/hooks/*.ts lists 12 hooks, matches expected count
- ISC-3: systemctl is-active qdrant.service returns "active", is-enabled returns "enabled"
- ISC-4: Screenshot shows login page with expected logo and title
- ISC-5: Test output: PASS test_user_authentication (0.23s)
```

**Why good:** Each shows exactly what was checked and what the result was.

---

⚠️ **Weak Evidence** (Vague, Not Reproducible)

```markdown
- ISC-1: Manually verified, looks good
- ISC-2: Tested and working
- ISC-3: Checked, seems fine
```

**Why weak:** No details on what was checked or how to reproduce.

---

❌ **Insufficient Evidence** (No Evidence)

```markdown
- ISC-1: Done
- ISC-2: Implemented
- ISC-3: Complete
```

**Why insufficient:** These are status updates, not verification.

---

### Evidence Types by Artifact

| Artifact Type | Required Evidence | Example |
|---------------|-------------------|---------|
| **Web page/UI** | Browser screenshot or curl output | `Interceptor screenshot shows dashboard with 3 charts` |
| **HTTP endpoint** | curl with status + body | `curl /api/users returns 200 with JSON array[3]` |
| **CLI tool** | Actual stdout | `./script.sh outputs "Success: 5 files processed"` |
| **Database write** | SELECT confirming write | `SELECT COUNT(*) FROM users WHERE role='admin' returns 2` |
| **File write** | Read confirming content | `cat config.json contains "api_key" field (value redacted)` |
| **Service** | systemctl status | `systemctl is-active nginx returns 'active'` |
| **Hook/skill** | Direct invocation | `bun hooks/PreToolUse.hook.ts test input passes` |
| **Configuration** | Grep/jq showing setting | `jq .autoCompact settings.json returns true` |
| **Absence/Security** | Negative grep | `rg 'password' git log returns no matches` |

---

## Using ISCVerifier Tool

### Basic Usage

```bash
# Check current ISA (auto-detects most recent WORK session)
bun PAI/Tools/ISCVerifier.ts

# Check specific ISA file
bun PAI/Tools/ISCVerifier.ts MEMORY/WORK/20260705-session/ISA.md

# Get JSON output (for scripts)
bun PAI/Tools/ISCVerifier.ts --format json

# Get verification checklist template
bun PAI/Tools/ISCVerifier.ts --checklist
```

### Example Output

**When verification is incomplete:**

```
═══ ISC Verification Report ═════════════════

ISA: 20260705-132243_usage-suggestions-into-harness
Total ISCs: 8
Verified: 6
Missing: 2
Pass Rate: 75.0%

─── Missing Verification ────────────────────

❌ ISC-2: Current `model:` state probed, not assumed
❌ ISC-8: No fabricated settings keys

─── Verified ISCs ───────────────────────────

✅ ISC-1: Every harness lever named with file/field
   Evidence: Bash awk over agents/*.md frontmatter...
   
[... 5 more verified ISCs ...]

⚠️  VERIFICATION INCOMPLETE

Add verification evidence to ## Verification section:

- ISC-2: [describe evidence that criterion is met]
- ISC-8: [describe evidence that criterion is met]
```

**When verification is complete:**

```
═══ ISC Verification Report ═════════════════

ISA: 20260704-113000_investigate-deepseek-dspark
Total ISCs: 3
Verified: 3
Missing: 0
Pass Rate: 100.0%

─── Verified ISCs ───────────────────────────

✅ ISC-1: Research agents agree on framework details
✅ ISC-2: Assessed against fleet facts  
✅ ISC-3: Caveat carried into report

✅ ALL ISCs VERIFIED
```

---

## Algorithm Integration

### When to Run ISCVerifier

**VERIFY Phase - Before marking `phase: complete`:**

```markdown
━━━ ✅ VERIFY ━━━ 6/7

1. Run all live probes per Rule 1
2. Complete deliverable compliance check
3. **Run ISCVerifier to ensure all ISCs addressed**
4. If any ISCs missing:
   a. Add verification evidence to ## Verification
   b. Re-run ISCVerifier
   c. Repeat until 100%
5. Mark phase: complete only when ISCVerifier exits 0
```

### Shell Integration Example

```bash
# In Algorithm VERIFY phase, after all other checks:

echo "Checking ISC verification completeness..."
if ! bun PAI/Tools/ISCVerifier.ts; then
  echo ""
  echo "⚠️  Cannot mark phase: complete - ISCs missing verification"
  echo ""
  bun PAI/Tools/ISCVerifier.ts --checklist
  exit 1
fi

echo "✅ All ISCs verified, safe to mark complete"
```

---

## Verification Workflow

### Step-by-Step Process

**1. Complete all work (BUILD/EXECUTE phase)**

**2. Run ISCVerifier to check status:**
```bash
bun PAI/Tools/ISCVerifier.ts
```

**3. If verification incomplete, get checklist:**
```bash
bun PAI/Tools/ISCVerifier.ts --checklist
```

**4. Copy checklist to ISA `## Verification` section (or enhance existing)**

**5. For each missing ISC, add concrete evidence:**

```markdown
## Verification

- ISC-1: curl localhost:3000 returns 200 OK with JSON {"status":"healthy"}
- ISC-2: ls hooks/ shows PreToolUse.hook.ts, PostToolUse.hook.ts (2 hooks)
- ISC-3: jq .permissions settings.json | wc -l returns 137 (all scoped)
```

**6. Re-run ISCVerifier to confirm:**
```bash
bun PAI/Tools/ISCVerifier.ts
```

**7. Repeat until 100% verified**

**8. Mark `phase: complete`**

---

## Common Patterns

### Grouped Verification

When multiple ISCs share the same evidence:

```markdown
- ISC-1/ISC-2/ISC-3: grep 'export' env.sh shows API_KEY, DB_URL, SMTP_HOST all present
```

ISCVerifier detects grouped format: `ISC-N/ISC-M:`

### Deferred Verification

For criteria that can't be verified immediately:

```markdown
## Criteria
- [DEFERRED-VERIFY] ISC-5: Cloudflare Worker responds after DNS propagation

## Verification
- ISC-5: DEFERRED - DNS propagation takes 24-48h, follow-up task: verify-cf-deploy-20260706
```

Mark checkbox as `[DEFERRED-VERIFY]` in Criteria section (per Algorithm doctrine).

### Negative Verification (Proving Absence)

For security/safety criteria:

```markdown
- ISC-3: rg 'password|api_key|secret' git log --all returns 0 matches
- ISC-7: biome check PAI/Tools/ returns 0 errors
```

---

## Troubleshooting

### "No ## Criteria section found"

**Problem:** ISA doesn't have a `## Criteria` section  
**Solution:** Add Criteria section with ISCs, or skip verification if ISA format doesn't use ISCs

### "No ISC items found"

**Problem:** Criteria section exists but no `- [ ] ISC-N:` format items  
**Solution:** Ensure ISCs follow format: `- [ ] ISC-1: Description`

### "ISC-N mentioned but not detected as verified"

**Problem:** ISC mentioned in Verification but ISCVerifier doesn't detect it  
**Solution:** Ensure format is `ISC-N:` or `ISC-N ` (with colon, space, comma, or slash)

**Valid formats:**
- `- ISC-1: Evidence here` ✅
- `ISC-1: Evidence` ✅  
- `ISC-1/ISC-2: Grouped` ✅
- `ISC-1, ISC-2: Listed` ✅

**Invalid formats:**
- `ISC-1 Evidence` ❌ (no punctuation)
- `The ISC-1 criterion` ❌ (mid-sentence, not evidence)

---

## Integration with PolicyCheck

PolicyCheck (`PAI/Tools/PolicyCheck.ts`) uses the same detection logic to measure ISC verification pass rate across recent ISAs.

**GOV-QUAL-001 - Verification Requirements:**
- **Target:** 95% pass rate across last 10 ISAs
- **Current:** Run `bun PAI/Tools/PolicyCheck.ts` to check
- **Remediation:** Use ISCVerifier to ensure each ISA reaches 100%

---

## Best Practices

### 1. Verify Incrementally

Don't wait until VERIFY phase to add evidence. As you complete each ISC during BUILD/EXECUTE, immediately add verification evidence.

**Good pattern:**
```markdown
## Criteria
- [x] ISC-1: Web server responds with 200
- [ ] ISC-2: Database has 3 backups
- [ ] ISC-3: Config file has API key

## Verification  
- ISC-1: curl localhost:8080 returns 200 OK (verified 2026-07-05 14:23)
```

### 2. Be Specific

Vague evidence doesn't help future debugging.

❌ `ISC-1: Tested, works`  
✅ `ISC-1: curl /api/health returns {"status":"ok","uptime":3600}`

### 3. Include Commands

Show exactly how to reproduce the verification.

✅ `ISC-2: systemctl is-active qdrant returns "active", is-enabled returns "enabled"`

### 4. Redact Secrets

If evidence contains sensitive data, redact it.

✅ `ISC-3: jq .api_key config.json returns "<REDACTED>" (non-null)`

### 5. Screenshot When Appropriate

For UI/visual criteria, describe what the screenshot shows.

✅ `ISC-5: Interceptor screenshot shows dashboard with 3 charts, logout button, username "admin"`

---

## FAQ

**Q: Do I need to verify every ISC?**  
A: Yes. Target is 100% for new work. PolicyCheck compliance requires 95%+ across recent ISAs.

**Q: What if verification is impossible?**  
A: Mark as `[DEFERRED-VERIFY]` in Criteria section and document follow-up task. See Algorithm VERIFY Rule 1 for probe-impossible escape clause.

**Q: Can I group ISCs in verification?**  
A: Yes. Use `ISC-1/ISC-2: Combined evidence` format.

**Q: What if ISCVerifier shows false positive/negative?**  
A: File issue. Detection logic in `PAI/Tools/PolicyCheck.ts` `checkVerificationRequirements()` function.

**Q: Do I run this on every session?**  
A: Run on sessions with ISAs (typically E2+ effort). E1 Standard quick tasks may not have ISAs.

---

## See Also

- **Algorithm VERIFY Phase:** `PAI/Algorithm/v5.7.11.md` Rule 1 (Live-Probe)
- **PolicyCheck:** `PAI/Tools/PolicyCheck.ts` GOV-QUAL-001
- **ISA Format:** `PAI/Algorithm/v5.7.11.md` ISA Quality System section
- **Troubleshooting Method:** `PAI/DOCUMENTATION/TROUBLESHOOTING_METHOD.md`

---

## Tool Implementation

**Source:** `PAI/Tools/ISCVerifier.ts` (400 lines)

**Detection Logic:**
1. Extract `## Criteria` section (handles sub-sections with `###`)
2. Parse all `- [ ] ISC-N:` items (unique IDs only)
3. Extract `## Verification` section  
4. Check each ISC-N is mentioned with format: `ISC-N[:\\s,/]`
5. Report verified vs missing

**Accuracy:** 100% detection on test ISAs (Phase 2 validation)

---

**Last Updated:** 2026-07-05  
**Version:** 1.0.0  
**Status:** Production Ready
