# PAI Interactivity — Quick Reference

**Full docs:** `INTERACTIVITY.md` | **Examples:** `INTERACTIVITY_EXAMPLES.md`

---

## PNC (Claude Code)

```typescript
// Complete work format FIRST
━━━ 📋 PLAN COMPLETE ━━━
[... work output ...]

// THEN ask
mcp_Question({
  questions: [{
    question: "Full question text?",
    header: "Short Label",
    options: [
      { label: "Option A", description: "Explanation" },
      { label: "Option B", description: "Explanation" }
    ],
    multiple: false  // true for multi-select
  }]
})

// Returns: user selection via UI
```

---

## PNK/PNX/PNG (CLI)

```bash
# Complete work format FIRST
━━━ 📋 PLAN COMPLETE ━━━
[... work output ...]

# THEN ask
bun ~/.claude/PAI/Tools/InteractivePrompt.ts ask \
  --question "Full question text?" \
  --options '[
    {"label":"Option A","description":"Explanation"},
    {"label":"Option B","description":"Explanation"}
  ]'

# Returns JSON:
# {"choice": 1, "label": "Option A", "custom": null}
# {"choice": -1, "label": null, "custom": "user typed text"}
```

---

## Common Patterns

### Security Override

```typescript
{
  question: "Security hook blocked [operation]. How proceed?",
  options: [
    { label: "Override Once", description: "Allow this operation only" },
    { label: "Override Session", description: "Allow this session" },
    { label: "Override Permanent", description: "Add to settings.json" },
    { label: "Cancel", description: "Stop operation" }
  ]
}
```

### Effort Confirmation

```typescript
{
  question: "Task is DETERMINED effort (~30min). Proceed?",
  options: [
    { label: "Yes (Recommended)", description: "Full ALGORITHM cycle" },
    { label: "Quick Mode", description: "Skip planning (risky)" },
    { label: "Simplify", description: "Clarify requirements first" }
  ]
}
```

### Model Selection

```typescript
{
  question: "Which analysis approach?",
  options: [
    { label: "Fast (Recommended)", description: "Ollama, zero cost, 2s" },
    { label: "Standard", description: "Claude Sonnet, ~8k tokens" },
    { label: "Council", description: "Multi-model, ~30k tokens" }
  ]
}
```

### Delegation Strategy

```typescript
{
  question: "Found N independent workstreams. Parallelize?",
  options: [
    { label: "Full Parallel", description: "N agents, fastest" },
    { label: "Sequential", description: "One by one, simpler" },
    { label: "Manual", description: "Explicit control" }
  ]
}
```

---

## Rules

1. **Complete work format FIRST** → then question
2. **Structured options** → never open-ended prose
3. **Describe consequences** → in every option description
4. **Recommend when appropriate** → mark preferred option "(Recommended)"
5. **Don't duplicate "Other"** → custom answer added automatically
6. **Use for security/cost decisions** → not routine choices

---

## Testing

```bash
# Test CLI
echo "1" | bun ~/.claude/PAI/Tools/InteractivePrompt.ts ask \
  --question "Test?" \
  --options '[{"label":"A","description":"First"}]'

# Run full test suite
cd ~/.claude/PAI/Tools
bun test InteractivePrompt.test.ts
```

---

## Per-Engine Status

| Engine | Implementation | Status |
|--------|---------------|---------|
| PNC | `mcp_Question` | ✅ Production |
| PNK | `InteractivePrompt.ts` | ✅ Production |
| PNX | `InteractivePrompt.ts` | ✅ Production |
| PNG | `InteractivePrompt.ts` | ✅ Production |
| PNO | N/A (never primary agent) | — |

---

## Anti-Patterns

### ❌ Don't

```
// Open-ended question
"What would you like me to do with this?"

// Asking before work complete
♻️ ALGORITHM MODE
[immediately call mcp_Question]

// Including "Other" option
options: [..., { label: "Other", description: "Something else" }]
```

### ✅ Do

```typescript
// Structured options
mcp_Question({
  questions: [{
    question: "What should I do with this code?",
    options: [
      { label: "Fix", description: "Add error handling" },
      { label: "Document", description: "Add TODO comment" },
      { label: "Ignore", description: "Not critical" }
    ]
  }]
})

// Work format first
━━━ 📋 PLAN COMPLETE ━━━
[... work ...]
[NOW call mcp_Question]

// Let platform add custom
options: [
  { label: "A", description: "..." },
  { label: "B", description: "..." }
  // "Custom" added automatically
]
```
