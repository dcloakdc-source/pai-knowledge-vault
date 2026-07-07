# Cross-Engine Budget System

**Version:** 1.0.0  
**Date:** 2026-07-05  
**Status:** Production Ready

## Overview

Unified budget tracking and cost-aware routing across all five PAI engines (PNC, PNG, PNO, PNX, PNK). Automatically routes tasks to free engines when budget is constrained.

## Architecture

```
┌─────────────────────────────────────────────────┐
│     CrossEngineBudget.ts                        │
│  (Unified tracking across 5 engines)            │
└───────────────┬─────────────────────────────────┘
                │
        ┌───────┴────────┐
        │                │
   ┌────▼────┐      ┌────▼────┐
   │ Budget  │      │ Engine  │
   │ Status  │      │ Router  │
   └────┬────┘      └────┬────┘
        │                │
        └────────┬───────┘
                 │
         ┌───────▼────────┐
         │ Routing Decision│
         │ (cost-aware)    │
         └─────────────────┘
```

## Components

### 1. CrossEngineBudget.ts

**Tracks:**
- Token usage per engine (input/output/cache)
- Cost per engine in USD
- Task types and session IDs
- 7-day rolling usage

**Functions:**
- `recordEngineUsage()` — Log usage to JSONL
- `getEngineUsage()` — Get usage summary by engine
- `checkBudget()` — Get current budget status + routing recommendation

### 2. EngineRouter Integration

**Budget-Aware Routing:**
- Automatically checks budget before routing
- Overrides cost sensitivity based on budget level
- Respects `can_use_paid` and `can_use_expensive` flags
- Falls back to free engines when budget constrained

### 3. Budget Thresholds

| Usage % | Recommended Routing | Behavior |
|---------|---------------------|----------|
| **0-50%** | `quality_first` | Use best engine for the task |
| **50-70%** | `balanced` | Balance cost and quality |
| **70-90%** | `free_first` | Prefer free, fallback to paid |
| **90-100%** | `free_only` | Force free engines only |

## Cost Per Engine

| Engine | Model | Input ($/Mtok) | Output ($/Mtok) | Total |
|--------|-------|----------------|-----------------|-------|
| **PNC** | claude-sonnet-4-5 | $3.00 | $15.00 | $18.00 |
| **PNG** | gemini-2.0-flash | $0.00 | $0.00 | **Free** |
| **PNO** | ollama (local) | $0.00 | $0.00 | **Free** |
| **PNX** | gpt-4o | $2.50 | $10.00 | $12.50 |
| **PNK** | claude-sonnet-4-5 | $3.00 | $15.00 | $18.00 |

**"Expensive" threshold:** >$10/Mtok combined (Claude/GPT-4)

## Usage

### CLI

```bash
# Check budget status
bun PAI/Tools/CrossEngineBudget.ts status
# {
#   "can_use_paid": true,
#   "can_use_expensive": true,
#   "reason": "Budget at 16.2% — use best engines",
#   "recommended_routing": "quality_first"
# }

# Record engine usage
bun PAI/Tools/CrossEngineBudget.ts record PNO 1000 500

# Usage report
bun PAI/Tools/CrossEngineBudget.ts report
# Shows requests, tokens, cost per engine

# Usage report (custom days)
bun PAI/Tools/CrossEngineBudget.ts report 30
```

### Programmatic

```typescript
import { recordEngineUsage, checkBudget } from "./CrossEngineBudget";

// Record usage after task completion
await recordEngineUsage("PNO", {
  model: "llama3.2:3b",
  input_tokens: 1000,
  output_tokens: 500,
  task_type: "fast_classification",
  session_id: "ses_abc123",
});

// Check budget before expensive operation
const budget = checkBudget();
if (!budget.can_use_expensive) {
  // Route to cheaper engine
}
```

### Integration with EngineRouter

```typescript
import { routeTask } from "./EngineRouter";

// Automatic budget-aware routing
const route = await routeTask("complex analysis");
// If budget > 70%, automatically routes to free engines
// If budget < 50%, routes to best engine
```

## Budget Status Fields

```typescript
interface BudgetStatus {
  can_use_paid: boolean;          // Can use any paid engine (PNC/PNX/PNK)
  can_use_expensive: boolean;     // Can use expensive engines (>$10/Mtok)
  reason?: string;                // Human-readable explanation
  
  window_usage: {
    pnc_output: number;           // PNC output tokens (5-hour window)
    total_output: number;         // Total output tokens
    limit: number;                // Configured limit
    percent: number;              // Usage percentage
  };
  
  weekly_usage: {
    total_output: number;         // Weekly output tokens
    limit: number;                // Configured weekly limit
    percent: number;              // Usage percentage
  };
  
  cost_7day: {
    pnc: number;                  // PNC cost (USD)
    pnx: number;                  // PNX cost (USD)
    pnk: number;                  // PNK cost (USD)
    total: number;                // Total cost (USD)
  };
  
  recommended_routing: string;    // "free_only" | "free_first" | "balanced" | "quality_first"
}
```

## Routing Behavior Examples

### Scenario 1: Budget at 15% (quality_first)

```bash
$ bun EngineRouter.ts route "research AI safety"
# → PNG (free, primary for research)

$ bun EngineRouter.ts route "complex architecture design"
# → PNC (paid, best for architecture)
```

**Behavior:** Uses best engine for the task, paid or free.

### Scenario 2: Budget at 65% (balanced)

```bash
$ bun EngineRouter.ts route "research AI safety"
# → PNG (free, primary)

$ bun EngineRouter.ts route "complex architecture design"
# → PNC (paid, but necessary for architecture)

$ bun EngineRouter.ts route "classify sentiment"
# → PNO (free, good enough for classification)
```

**Behavior:** Prefers free, uses paid when significant quality difference.

### Scenario 3: Budget at 75% (free_first)

```bash
$ bun EngineRouter.ts route "research AI safety"
# → PNG (free, primary)

$ bun EngineRouter.ts route "complex architecture design"
# → PNG (free fallback! Architecture quality degraded but saves budget)

$ bun EngineRouter.ts route "classify sentiment"
# → PNO (free)
```

**Behavior:** Strongly prefers free engines, paid only if absolutely necessary.

### Scenario 4: Budget at 95% (free_only)

```bash
$ bun EngineRouter.ts route "research AI safety"
# → PNG (free)

$ bun EngineRouter.ts route "complex architecture design"
# → PNG (free fallback, quality severely degraded)
# OR → Error/warning if no free engine can handle it

$ bun EngineRouter.ts route "classify sentiment"
# → PNO (free)
```

**Behavior:** **Forces free engines only**. Paid engines blocked.

## Cost Calculation

```typescript
// Example: PNC usage
const input_tokens = 5000;
const output_tokens = 2500;
const cache_read_tokens = 1000;
const cache_create_tokens = 500;

// Costs ($/Mtok)
const input_cost = (5000 / 1_000_000) * 3.00 = $0.015
const output_cost = (2500 / 1_000_000) * 15.00 = $0.0375
const cache_read_cost = (1000 / 1_000_000) * 0.30 = $0.0003
const cache_create_cost = (500 / 1_000_000) * 3.75 = $0.001875

// Total
const total_cost = $0.054675
```

**Cache pricing:**
- `cache_read`: ~10% of input price
- `cache_create`: ~1.25x input price

## Files

```
PAI/Tools/
├── CrossEngineBudget.ts        # Cross-engine budget tracker
├── EngineRouter.ts             # Smart router with budget integration
├── budget-shared.ts            # PNC budget system (existing)
└── engine-capability-matrix.json

~/MEMORY/STATE.claude/
├── engine-usage.jsonl          # Cross-engine usage log
├── token-budget.jsonl          # PNC usage log (existing)
└── token-budget-config.json    # Budget configuration

PAI/DOCUMENTATION/
├── CROSS_ENGINE_BUDGET.md      # This file
└── ENGINE_ORCHESTRATION.md     # Router documentation
```

## Configuration

Edit `~/MEMORY/STATE.claude/token-budget-config.json`:

```json
{
  "window_hours": 5,
  "window_output_limit": 8846779,
  "weekly_output_limit": 65000000
}
```

- **window_hours:** Claude's rolling enforcement window (always 5)
- **window_output_limit:** Output tokens per 5-hour window
- **weekly_output_limit:** Soft weekly ceiling (informational)

## Integration Points

### Algorithm OBSERVE Phase

```typescript
// Before expensive Algorithm run
const budget = checkBudget();
if (budget.window_usage.percent > 80) {
  // Present budget warning via mcp_Question
  // "Budget at 85% — proceed with full Algorithm (paid) or simplified (free)?"
}
```

### BudgetGuard Hook

```typescript
// In hooks/BudgetGuard.hook.ts
const budget = checkCrossEngineBudget();
if (!budget.can_use_paid) {
  // Block Task tool spawns to paid engines
  // Inject recommendation: "Budget critical — use Ollama for classification"
}
```

### Task Delegation

```typescript
// Before spawning multiple agents
const budget = checkBudget();
if (budget.recommended_routing === "free_only") {
  // Route all subtasks to PNO/PNG
  // Warn user: "Using free engines due to budget constraints"
}
```

## Monitoring

### Real-Time Dashboard

```
╔════════════════════════════════════════════════╗
║ PAI Budget Status                              ║
╠════════════════════════════════════════════════╣
║ Window (5h):  821k / 8.8M  (9.3%)   ✓ healthy ║
║ Weekly:       10.5M / 65M  (16.2%)  ✓ healthy ║
║                                                ║
║ 7-Day Cost:                                    ║
║   PNC: $0.0525                                 ║
║   PNX: $0.0000                                 ║
║   PNK: $0.0000                                 ║
║   Total: $0.0525                               ║
║                                                ║
║ Routing: quality_first                         ║
╚════════════════════════════════════════════════╝
```

### Alerts

**70% threshold:**
```
⚠️  Budget at 72% — routing switched to free_first
   Tasks will prefer PNG/PNO over PNC/PNX/PNK
```

**90% threshold:**
```
🚨 Budget CRITICAL at 93% — free engines only
   All paid engine routing blocked
   Consider upgrading plan or waiting for window reset
```

## Testing

```bash
# Simulate usage
cd PAI/Tools

# Record some usage
bun CrossEngineBudget.ts record PNO 1000 500
bun CrossEngineBudget.ts record PNC 5000 2500
bun CrossEngineBudget.ts record PNX 3000 1500

# Check budget impact
bun CrossEngineBudget.ts status

# View report
bun CrossEngineBudget.ts report

# Test routing
bun EngineRouter.ts route "complex task"
```

## Future Enhancements

### Planned (v1.1)
1. **Auto-record from hooks** — BudgetTracker.hook.ts records to engine-usage.jsonl
2. **PNG/PNX usage tracking** — Currently only PNC tracked automatically
3. **Budget notifications** — Voice alerts at thresholds
4. **Cost forecasting** — Predict when budget will hit limit
5. **Per-project budgets** — Separate budgets for different work

### Considered (v2.0)
6. **Smart caching** — Aggressive cache to reduce costs near limit
7. **Task queueing** — Defer non-urgent tasks until budget resets
8. **Cost optimization** — Suggest refactoring to reduce token usage
9. **Multi-user budgets** — Per-user tracking (if PAI goes multi-user)
10. **Budget alerts via Telegram** — Real-time mobile notifications

## Troubleshooting

### Budget shows 0% but I've been using PNC

**Cause:** window_output_limit is 0 (uncalibrated)  
**Fix:** Run `bun PAI/Tools/BudgetReport.ts --set-window-limit <limit>`

### Routing ignores budget

**Cause:** Budget check throwing error  
**Fix:** Check `engine-usage.jsonl` exists and is valid JSON lines

### Cost calculation seems wrong

**Cause:** Engine costs out of date in ENGINE_COSTS map  
**Fix:** Update costs in CrossEngineBudget.ts to match current pricing

### Free engines still used when budget < 50%

**Cause:** Free engines are primary for those task types (working as designed)  
**Fix:** This is correct — free engines used when they're best for the task

## Related Documentation

- `ENGINE_ORCHESTRATION.md` — Engine routing system
- `budget-shared.ts` — PNC budget tracking
- `BudgetGuard.hook.ts` — Budget enforcement hook
- `INTERACTIVITY.md` — User prompts for budget decisions

## Changelog

### 1.0.0 (2026-07-05)
- Initial cross-engine budget system
- Usage tracking for all 5 engines
- Cost calculation per engine
- Budget-aware routing integration
- 4-tier threshold system (critical/high/medium/low)
- CLI interface for status/record/report
- Programmatic API for integrations
