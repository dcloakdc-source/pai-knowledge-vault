# PAI Interactivity System — Examples

**Reference:** See `INTERACTIVITY.md` for full documentation.

This file contains real-world examples of using the interactivity system across all engines.

---

## Security Override (PNC)

**Context:** User tries to delete a sensitive directory, security hook blocks it.

```typescript
// PNC native implementation
mcp_Question({
  questions: [{
    question: "Security hook blocked: rm -rf ~/MEMORY/WORK/2026-07-05-security-audit. This contains 3 days of work. How should we proceed?",
    header: "Security Override",
    options: [
      {
        label: "Override Once",
        description: "Allow this single rm -rf operation only"
      },
      {
        label: "Override Session",
        description: "Allow rm operations in ~/MEMORY/WORK/ for this session"
      },
      {
        label: "Override Permanent",
        description: "Add Write(~/MEMORY/WORK/*) to settings.json permanently"
      },
      {
        label: "Cancel (Recommended)",
        description: "Stop and review what's being deleted first"
      }
    ]
  }]
})
```

**User selects:** Cancel  
**Action:** Stop operation, show user what would be deleted, suggest archive instead

---

## Security Override (PNK/PNX/PNG)

**Context:** Same scenario, but in CLI-based engine.

```bash
bun ~/.claude/PAI/Tools/InteractivePrompt.ts ask \
  --question "Security hook blocked: rm -rf ~/MEMORY/WORK/2026-07-05-security-audit. This contains 3 days of work. How should we proceed?" \
  --options '[
    {
      "label":"Override Once",
      "description":"Allow this single rm -rf operation only"
    },
    {
      "label":"Override Session",
      "description":"Allow rm operations in ~/MEMORY/WORK/ for this session"
    },
    {
      "label":"Override Permanent",
      "description":"Add Write(~/MEMORY/WORK/*) to settings.json permanently"
    },
    {
      "label":"Cancel (Recommended)",
      "description":"Stop and review what is being deleted first"
    }
  ]'
```

**Renders as:**
```
╔══════════════════════════════════════════════════════════╗
║ PAI INTERACTIVE PROMPT                                   ║
╠══════════════════════════════════════════════════════════╣
║ Security hook blocked: rm -rf                            ║
║ ~/MEMORY/WORK/2026-07-05-security-audit. This contains  ║
║ 3 days of work. How should we proceed?                   ║
║                                                          ║
║  1. Override Once                                        ║
║     Allow this single rm -rf operation only            ║
║                                                          ║
║  2. Override Session                                     ║
║     Allow rm operations in ~/MEMORY/WORK/ for this     ║
║     session                                            ║
║                                                          ║
║  3. Override Permanent                                   ║
║     Add Write(~/MEMORY/WORK/*) to settings.json        ║
║     permanently                                        ║
║                                                          ║
║  4. Cancel (Recommended)                                 ║
║     Stop and review what is being deleted first        ║
║                                                          ║
║  5. Custom (type your own answer)                        ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝

Enter choice (1-5 or text): 
```

**User enters:** 4  
**Returns:** `{"choice": 4, "label": "Cancel (Recommended)", "custom": null}`  
**Action:** Same as PNC — stop and review

---

## Effort Tier Confirmation (PNC)

**Context:** OBSERVE phase detects DETERMINED effort (~30min, multi-file work).

```typescript
// After ISC extraction
♻️ ALGORITHM MODE
🗒️ TASK: Implement authentication system
⏳ Entering OBSERVE…

[... ISC extraction ...]

═══ OBSERVE COMPLETE ═══
Effort: DETERMINED (~30min)
ISC: 18 criteria
Files: 5 estimated

// NOW ask for confirmation
mcp_Question({
  questions: [{
    question: "This task is DETERMINED effort (~30min, 5 files). Proceed with full ALGORITHM execution?",
    header: "Effort Confirmation",
    options: [
      {
        label: "Yes (Recommended)",
        description: "Run full OBSERVE→THINK→PLAN→BUILD→VERIFY cycle with ISC tracking"
      },
      {
        label: "Quick Mode",
        description: "Skip THINK/PLAN, go straight to BUILD (risks missing edge cases)"
      },
      {
        label: "Simplify Task",
        description: "Let me clarify what's actually needed first"
      }
    ]
  }]
})
```

**User selects:** Yes  
**Action:** Continue to THINK phase

---

## Model Selection (PNC)

**Context:** User asks to "analyze these 50 log files for errors"

```typescript
mcp_Question({
  questions: [{
    question: "I can analyze these 50 log files using different approaches. Which do you prefer?",
    header: "Analysis Method",
    options: [
      {
        label: "Fast (Recommended)",
        description: "Ollama classification, zero cost, ~10 seconds, 90% accuracy"
      },
      {
        label: "Standard",
        description: "Claude Sonnet, ~15k tokens ($0.15), ~45 seconds, 98% accuracy"
      },
      {
        label: "Deep",
        description: "Multi-model council, ~60k tokens ($0.60), ~3 minutes, consensus-based"
      }
    ]
  }]
})
```

**User selects:** Fast  
**Action:** Route to Ollama via `Skill("OllamaSkill", "classify these 50 log entries...")`

---

## Delegation Strategy (PNC)

**Context:** PLAN phase identifies 4 independent workstreams.

```typescript
━━━ 📋 PLAN ━━━

Deliverable Manifest:
1. Auth endpoints (backend/routes/auth.ts)
2. Token middleware (backend/middleware/jwt.ts)
3. Frontend auth context (frontend/contexts/AuthContext.tsx)
4. Login UI (frontend/pages/Login.tsx)

Parallelism: All 4 independent, no shared state

mcp_Question({
  questions: [{
    question: "Found 4 independent workstreams. How should we parallelize?",
    header: "Delegation Strategy",
    options: [
      {
        label: "Full Parallel (Recommended)",
        description: "4 haiku agents in background, ~2 min total, automatic merge"
      },
      {
        label: "Sequential",
        description: "One at a time in main context, ~8 min total, simpler"
      },
      {
        label: "Manual",
        description: "Let me handle delegation explicitly per workstream"
      }
    ]
  }]
})
```

**User selects:** Full Parallel  
**Action:** Spawn 4 background agents via Task tool

---

## Research Depth Selection (PNG)

**Context:** User asks "research Cloudflare Workers security best practices"

```bash
🗒️ TASK: Research Cloudflare Workers security

bun ~/.claude/PAI/Tools/InteractivePrompt.ts ask \
  --question "How deep should this research go?" \
  --options '[
    {
      "label":"Quick (Recommended)",
      "description":"Single query, official docs, ~30 seconds"
    },
    {
      "label":"Standard",
      "description":"3 queries (docs + community + examples), ~2 minutes"
    },
    {
      "label":"Extensive",
      "description":"Multi-engine fan-out (Gemini + Perplexity + ArXiv), ~5 minutes"
    }
  ]'
```

**User enters:** 1  
**Returns:** `{"choice": 1, "label": "Quick (Recommended)", "custom": null}`  
**Action:** Single `agy` query to Cloudflare docs

---

## Verification Gate (PNX)

**Context:** Cato adversarial review finds potential issues.

```bash
━━━ ⚡ CATO REVIEW ━━━
Issues found: 3
- Missing null check in getUserProfile (line 42)
- Unhandled promise rejection in auth middleware
- TODO comment from 2 months ago

bun ~/.claude/PAI/Tools/InteractivePrompt.ts ask \
  --question "Cato review found 3 issues. How should we proceed?" \
  --options '[
    {
      "label":"Pass with Notes",
      "description":"Issues are non-blocking, document for follow-up"
    },
    {
      "label":"Investigate",
      "description":"Deep dive on each issue before approving"
    },
    {
      "label":"Fail",
      "description":"Block this PR, require fixes before merge"
    }
  ]'
```

**User enters:** 2  
**Returns:** `{"choice": 2, "label": "Investigate", "custom": null}`  
**Action:** Spawn investigation subagent per issue

---

## Custom Answer Example (Any Engine)

**Context:** User has a specific preference not covered by options.

**PNC:**
```typescript
mcp_Question({
  questions: [{
    question: "Which test framework should we use?",
    header: "Test Framework",
    options: [
      { label: "Vitest", description: "Fast, Vite-native, recommended" },
      { label: "Jest", description: "Industry standard, mature" },
      { label: "Bun Test", description: "Native to Bun runtime" }
    ]
  }]
})
```

**User types:** "playwright test"  
**Returns:** `{"choice": -1, "label": null, "custom": "playwright test"}`  
**Action:** Use Playwright Test (not in original options)

**CLI equivalent:**
```bash
bun ~/.claude/PAI/Tools/InteractivePrompt.ts ask \
  --question "Which test framework?" \
  --options '[
    {"label":"Vitest","description":"Fast, Vite-native"},
    {"label":"Jest","description":"Industry standard"},
    {"label":"Bun Test","description":"Native to Bun"}
  ]'
```

**User enters:** playwright test  
**Returns:** `{"choice": -1, "label": null, "custom": "playwright test"}`  
**Action:** Same — use Playwright Test

---

## Multi-Select Example (PNC)

**Context:** User needs to select multiple code quality checks.

```typescript
mcp_Question({
  questions: [{
    question: "Which code quality checks should I run?",
    header: "Quality Checks",
    multiple: true, // ← Enable multi-select
    options: [
      { label: "TypeScript", description: "Type checking with tsc" },
      { label: "Linting", description: "ESLint rules" },
      { label: "Formatting", description: "Prettier format check" },
      { label: "Tests", description: "Run test suite" },
      { label: "Security", description: "npm audit + Snyk scan" }
    ]
  }]
})
```

**User selects:** 1, 2, 4  
**Returns:** Array of selections  
**Action:** Run TypeScript, Linting, and Tests (skip Formatting and Security)

**Note:** Multi-select is currently a PNC-only feature (mcp_Question supports it). CLI implementation (PNK/PNX/PNG) would need enhancement to parse comma-separated input.

---

## Anti-Pattern: Open-Ended Question

### ❌ Wrong

```
I found an issue in the code. What would you like me to do about it?
Should I fix it, document it, or ignore it? Let me know!
```

**Why wrong:** Forces user to type full prose response.

### ✅ Correct

```typescript
mcp_Question({
  questions: [{
    question: "Found issue: potential null pointer in getUserById. What should I do?",
    header: "Issue Action",
    options: [
      { label: "Fix", description: "Add null check and error handling" },
      { label: "Document", description: "Add TODO comment for later" },
      { label: "Ignore", description: "Not critical, skip for now" }
    ]
  }]
})
```

---

## Anti-Pattern: Asking Before Work Complete

### ❌ Wrong

```
♻️ ALGORITHM MODE
🗒️ TASK: Implement auth

[immediately calls mcp_Question without completing OBSERVE]
```

**Why wrong:** Violates "response format before questions" rule.

### ✅ Correct

```
♻️ ALGORITHM MODE
🗒️ TASK: Implement auth
⏳ Entering OBSERVE…

[... complete OBSERVE phase ...]

━━━ 👁️ OBSERVE COMPLETE ━━━
Effort: DETERMINED
ISC: 16 criteria

[NOW ask for confirmation]
mcp_Question({ ... })
```

---

## Testing Your Implementation

### PNC (Native)

1. Open Claude Code
2. Invoke any question pattern from above
3. Verify UI renders with options
4. Select option
5. Verify response includes selected choice

### PNK/PNX/PNG (CLI)

```bash
# Test basic functionality
echo "1" | bun ~/.claude/PAI/Tools/InteractivePrompt.ts ask \
  --question "Test?" \
  --options '[{"label":"A","description":"First"}]'

# Expected output:
# [formatted prompt]
# {"choice": 1, "label": "A", "custom": null}

# Test custom answer
echo "something custom" | bun ~/.claude/PAI/Tools/InteractivePrompt.ts ask \
  --question "Test?" \
  --options '[{"label":"A","description":"First"}]'

# Expected output:
# [formatted prompt]
# {"choice": -1, "label": null, "custom": "something custom"}

# Test fuzzy matching
echo "fast" | bun ~/.claude/PAI/Tools/InteractivePrompt.ts ask \
  --question "Which?" \
  --options '[{"label":"Fast","description":"Quick"},{"label":"Slow","description":"Thorough"}]'

# Expected output:
# [formatted prompt]
# {"choice": 1, "label": "Fast", "custom": null}
```

### Full Test Suite

```bash
cd ~/.claude/PAI/Tools
bun test InteractivePrompt.test.ts
# Should see: 5 pass, 0 fail
```

---

## Integration with Existing Workflows

### Algorithm OBSERVE Phase

```
━━━ 👁️ OBSERVE ━━━
[... ISC extraction ...]

# If DETERMINED+ effort:
mcp_Question({ ... effort confirmation ... })
```

### Security Hook Trigger

```
[PAI SECURITY] 🚨 BLOCKED
Operation: Write(~/sensitive-file)

# Read security-pending.json
# Present override options via mcp_Question or InteractivePrompt.ts
```

### Delegation Decision

```
━━━ 📋 PLAN ━━━
[... identify workstreams ...]

# If 3+ parallel workstreams:
mcp_Question({ ... delegation strategy ... })
```

### Model Routing

```
[Classification task detected]

# Before expensive operation:
mcp_Question({ ... model selection ... })
```

---

## Related Documentation

- `INTERACTIVITY.md` — Full system documentation
- `CLAUDE.md` — PNC-specific rules
- `AGENTS.md` (per-engine) — Engine-specific interactivity sections
- `PAI/DOCUMENTATION/SECURITY_OVERRIDE_MECHANISM.md` — Security override internals

---

## Changelog

### 1.0.0 (2026-07-05)
- Initial examples collection
- Security override examples (PNC + CLI)
- Effort confirmation example
- Model selection example
- Delegation strategy example
- Research depth selection example (PNG)
- Verification gate example (PNX)
- Custom answer examples
- Multi-select example (PNC only)
- Anti-pattern examples
- Testing procedures
