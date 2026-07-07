# PAI Cross-Engine Interactivity System

**Version:** 1.0.0  
**Date:** 2026-07-05  
**Status:** Production

## Overview

PAI's interactivity system provides a unified interface for asking structured questions across all five engines (PNC, PNG, PNO, PNX, PNK), with platform-appropriate implementations.

## Core Principles

1. **Response format before questions** — Complete the work output (ALGORITHM/NATIVE/MINIMAL format) FIRST, then invoke the question mechanism
2. **Structured over freeform** — Use structured options with descriptions, not open-ended prose questions
3. **Platform-appropriate degradation** — Native UI tools where available, formatted CLI prompts elsewhere
4. **Security-critical interactivity** — Override prompts for destructive operations with clear consequence descriptions

## Question Structure

All questions follow this schema:

```typescript
interface Question {
  question: string;          // Full question text
  header: string;            // Short label (max 30 chars)
  options: Option[];         // Available choices
  multiple?: boolean;        // Allow selecting multiple (default: false)
}

interface Option {
  label: string;            // Display text (1-5 words, concise)
  description: string;      // Explanation of choice
}
```

## Per-Engine Implementation

### PNC (Claude Code) — Native `mcp_Question`

**Platform capability:** Full structured question UI with rich formatting

**Usage:**
```typescript
mcp_Question({
  questions: [{
    question: "Which approach should we take?",
    header: "Analysis Method",
    options: [
      { 
        label: "Fast (Recommended)", 
        description: "Ollama classification, zero cost, 2s runtime" 
      },
      { 
        label: "Deep", 
        description: "Claude Sonnet analysis, ~8k tokens, comprehensive" 
      },
      { 
        label: "Council", 
        description: "Multi-model debate, 4 engines, ~30k tokens total" 
      }
    ],
    multiple: false
  }]
})
```

**Features:**
- Automatic "Type your own answer" option when `custom: true` (default)
- Multi-select support
- Rich descriptions
- First option with "(Recommended)" suffix = suggested choice

**Rules:**
- Always complete ALGORITHM/NATIVE/MINIMAL format output first
- Then invoke `mcp_Question` at end
- Never ask open-ended "what do you want?" questions
- Don't include "Other" or catch-all options (handled by `custom`)

### PNK (OpenCode) — CLI Prompt Bridge

**Platform capability:** Terminal-based, no native structured UI

**Usage:**
```bash
bun ~/.claude/PAI/Tools/InteractivePrompt.ts ask \
  --question "Which approach should we take?" \
  --options '[
    {"label":"Fast","description":"Ollama classification, zero cost"},
    {"label":"Deep","description":"Sonnet analysis, ~8k tokens"}
  ]'
```

**Implementation:** See `PAI/Tools/InteractivePrompt.ts`

**Renders as:**
```
╔═══════════════════════════════════════════════════╗
║ PAI INTERACTIVE PROMPT                            ║
╠═══════════════════════════════════════════════════╣
║ Which approach should we take?                    ║
║                                                   ║
║  1. Fast                                          ║
║     Ollama classification, zero cost              ║
║                                                   ║
║  2. Deep                                          ║
║     Sonnet analysis, ~8k tokens                   ║
║                                                   ║
║  3. Custom (type your own answer)                 ║
║                                                   ║
╚═══════════════════════════════════════════════════╝

Enter choice (1-3 or text): 
```

**Parsing:**
- Numeric: maps to option index
- Text: fuzzy match against labels, else treat as custom answer
- Returns JSON: `{"choice": 1, "label": "Fast", "custom": null}`

### PNG (Antigravity/Gemini) — CLI Prompt

**Platform capability:** CLI-only, runs via `agy` command or Gemini Code Assist

**Usage:** Same as PNK — uses `InteractivePrompt.ts`

```bash
# From PNG context
bun ~/.claude/PAI/Tools/InteractivePrompt.ts ask \
  --question "..." \
  --options '[...]'
```

**Context:** PNG is primarily used for web research fan-out and factual lookup, so interactive prompts are rare. When needed, uses the same CLI bridge as PNK.

### PNX (Codex CLI) — CLI Prompt

**Platform capability:** CLI-only, terminal-based

**Usage:** Same as PNK/PNG — uses `InteractivePrompt.ts`

```bash
# From PNX context
bun ~/.claude/PAI/Tools/InteractivePrompt.ts ask \
  --question "..." \
  --options '[...]'
```

**Context:** PNX is primarily used for adversarial verification (Cato) and high-reasoning code generation (Forge). Interactive prompts mainly for verification gates and override decisions.

### PNO (Ollama) — No Interactivity

**Platform capability:** None — stateless inference API

**Usage:** N/A — PNO is never the primary agent

**Context:** Ollama is a capability layer called BY other engines for classification, JSON extraction, summarization. It never prompts users directly.

## Common Patterns

### Security Override Protocol

When a tool call is blocked, present three options with consequences:

```typescript
// PNC
mcp_Question({
  questions: [{
    question: "Security hook blocked this operation. How should we proceed?",
    header: "Security Override",
    options: [
      {
        label: "Override Once",
        description: "Allow this operation for this single invocation only"
      },
      {
        label: "Override Session",
        description: "Allow for this session (until restart)"
      },
      {
        label: "Override Permanent",
        description: "Add to settings.json allow list (affects all future sessions)"
      },
      {
        label: "Cancel",
        description: "Do not proceed with this operation"
      }
    ]
  }]
})

// PNK/PNG/PNX (CLI)
bun ~/.claude/PAI/Tools/InteractivePrompt.ts ask \
  --question "Security hook blocked: rm -rf /sensitive. How should we proceed?" \
  --options '[
    {"label":"Override Once","description":"Allow this operation once"},
    {"label":"Override Session","description":"Allow this session only"},
    {"label":"Override Permanent","description":"Add to settings.json permanently"},
    {"label":"Cancel","description":"Do not proceed"}
  ]'
```

**Rules:**
- Always describe the blocked operation explicitly
- Always include consequences in descriptions
- Always offer Cancel option
- Read `~/MEMORY/STATE.claude/security-pending.json` for exact blocked operation details

### Effort Tier Confirmation

At OBSERVE phase when effort is DETERMINED or higher:

```typescript
// After ISC extraction
mcp_Question({
  questions: [{
    question: "This task is DETERMINED effort (~30min, multi-file). Proceed with full ALGORITHM execution?",
    header: "Effort Confirmation",
    options: [
      {
        label: "Yes (Recommended)",
        description: "Run full OBSERVE→THINK→PLAN→BUILD→VERIFY cycle"
      },
      {
        label: "Quick Mode",
        description: "Skip to BUILD, minimal planning (risks rework)"
      },
      {
        label: "Cancel",
        description: "Stop and clarify requirements"
      }
    ]
  }]
})
```

### Model Selection

When routing to expensive resources:

```typescript
mcp_Question({
  questions: [{
    question: "This analysis can use multiple approaches. Which do you prefer?",
    header: "Analysis Approach",
    options: [
      {
        label: "Fast (Recommended)",
        description: "Ollama classification, zero cost, 2s runtime"
      },
      {
        label: "Standard",
        description: "Claude Sonnet, ~8k tokens, comprehensive"
      },
      {
        label: "Council",
        description: "Multi-model debate, 4 engines, ~30k tokens"
      }
    ]
  }]
})
```

### Delegation Strategy

When multiple parallel approaches exist:

```typescript
mcp_Question({
  questions: [{
    question: "Found 3 independent workstreams. How should we parallelize?",
    header: "Delegation Strategy",
    options: [
      {
        label: "Full Parallel (Recommended)",
        description: "3 haiku agents in background, fastest"
      },
      {
        label: "Sequential",
        description: "One at a time in main context, slower but simpler"
      },
      {
        label: "Manual",
        description: "Let me handle delegation explicitly"
      }
    ],
    multiple: false
  }]
})
```

## Anti-Patterns

### ❌ Don't: Open-ended prose questions

```
"What would you like me to do with this? I could analyze it, 
or perhaps refactor it, or maybe just document it. Let me know!"
```

**Why wrong:** Forces user to type full response, no structured choices

### ✅ Do: Structured options

```typescript
mcp_Question({
  questions: [{
    question: "What should we do with this code?",
    header: "Code Action",
    options: [
      { label: "Analyze", description: "Review for issues and improvements" },
      { label: "Refactor", description: "Restructure for better quality" },
      { label: "Document", description: "Add comments and README" }
    ]
  }]
})
```

### ❌ Don't: Asking before completing work format

```
♻️ ALGORITHM MODE
🗒️ TASK: Analyze security

[immediately calls mcp_Question without completing OBSERVE]
```

**Why wrong:** Violates "response format before questions" rule

### ✅ Do: Complete format, then ask

```
♻️ ALGORITHM MODE
🗒️ TASK: Analyze security
⏳ Entering OBSERVE…

[completes OBSERVE phase with ISC extraction]

═══ OBSERVE COMPLETE ═══
ISC: 12 criteria extracted
Effort: DETERMINED (~30min)

[NOW call mcp_Question for confirmation]
```

### ❌ Don't: Including "Other" option when custom is enabled

```typescript
options: [
  { label: "Option A", description: "..." },
  { label: "Option B", description: "..." },
  { label: "Other", description: "Something else" }  // ❌ Redundant
]
```

**Why wrong:** `mcp_Question` adds "Type your own answer" automatically when `custom: true` (default)

### ✅ Do: Let the platform handle custom

```typescript
options: [
  { label: "Option A", description: "..." },
  { label: "Option B", description: "..." }
  // Platform adds "Type your own answer" automatically
]
```

## Testing

### Test PNC Implementation

```bash
# From Claude Code context
mcp_Question({
  questions: [{
    question: "Test question?",
    header: "Test",
    options: [
      { label: "A", description: "First option" },
      { label: "B", description: "Second option" }
    ]
  }]
})
```

### Test CLI Implementation

```bash
# Test InteractivePrompt.ts
bun ~/.claude/PAI/Tools/InteractivePrompt.ts ask \
  --question "Which engine should handle this?" \
  --options '[
    {"label":"PNC","description":"Claude Code, full TUI"},
    {"label":"PNK","description":"OpenCode, multi-model"},
    {"label":"PNX","description":"Codex, autonomous"}
  ]'

# Expected output: formatted prompt, reads stdin, returns JSON
```

## Migration Notes

**Pre-1.0.0 behavior:**
- No structured question system for PNK/PNG/PNX
- Open-ended prose questions in all engines
- Inconsistent security override prompts
- No documented patterns

**Post-1.0.0 behavior:**
- Unified question interface across all engines
- Platform-appropriate implementations (native UI vs CLI)
- Security Override Protocol standardized
- Common patterns documented and enforced

## Related Documentation

- `CLAUDE.md` — PNC configuration and rules
- `AGENTS.md` (per-engine) — Engine-specific bridge instructions
- `PAI/DOCUMENTATION/SECURITY_OVERRIDE_MECHANISM.md` — Security override internals
- `PAI/Tools/InteractivePrompt.ts` — CLI implementation

## Changelog

### 1.0.0 (2026-07-05)
- Initial documentation of cross-engine interactivity system
- Documented PNC native `mcp_Question` usage
- Created `InteractivePrompt.ts` CLI bridge for PNK/PNG/PNX
- Established common patterns (Security Override, Effort Confirmation, Model Selection, Delegation)
- Anti-patterns section
- Testing procedures
