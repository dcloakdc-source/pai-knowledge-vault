# ISC Verification Checkpoint

**Purpose:** VERIFY phase quality gate - ensure all ISCs have verification evidence before marking complete.

**When:** After completing VERIFY Rule 1-4 checks, before marking `phase: complete`.

---

## Quick Check

```bash
bun PAI/Tools/ISCVerifier.ts
```

**Exit codes:**
- `0` - All ISCs verified (100%) → Safe to mark complete ✅
- `1` - Missing verification → **Do not mark complete** ⚠️
- `2` - Error (no ISA or invalid format)

---

## If Verification Incomplete

**1. Get checklist:**
```bash
bun PAI/Tools/ISCVerifier.ts --checklist
```

**2. Add evidence to `## Verification` section:**

```markdown
## Verification

- ISC-1: [concrete evidence - command output, screenshot description, test result]
- ISC-2: [concrete evidence - be specific, show what was checked]
```

**3. Re-check until 100%:**
```bash
bun PAI/Tools/ISCVerifier.ts
```

---

## Evidence Standards

✅ **Good:** `curl localhost:8080 returns 200 OK with JSON {"status":"healthy"}`  
⚠️ **Weak:** `Manually verified, looks good`  
❌ **Insufficient:** `Done`

See `PAI/DOCUMENTATION/ISC_VERIFICATION_GUIDE.md` for complete standards and examples.

---

## Integration Pattern

```bash
# In VERIFY phase, after all other checks:

if ! bun PAI/Tools/ISCVerifier.ts; then
  echo "⚠️  ISC verification incomplete - cannot mark complete"
  bun PAI/Tools/ISCVerifier.ts --checklist
  # Add evidence, then re-check
else
  echo "✅ All ISCs verified"
  # Proceed to mark phase: complete
fi
```

---

**Tool:** `PAI/Tools/ISCVerifier.ts`  
**Full Guide:** `PAI/DOCUMENTATION/ISC_VERIFICATION_GUIDE.md`  
**PolicyCheck:** GOV-QUAL-001 (target: 95% across ISAs)
