# PAI Engine Orchestration System

**Version:** 1.0.0  
**Date:** 2026-07-05  
**Status:** Production Ready

## Overview

Smart cross-engine task routing with health checks, failover, and cost-aware dispatching across all five PAI engines (PNC, PNG, PNO, PNX, PNK).

## Components

### 1. Engine Capability Matrix (`engine-capability-matrix.json`)

Defines each engine's:
- Capabilities (what it can do)
- Health check method
- Cost structure
- Rate limits
- Routing rules

### 2. EngineRouter Tool (`EngineRouter.ts`)

Smart router that:
- Classifies tasks from natural language
- Matches tasks to optimal engines
- Checks engine health before routing
- Provides failover to secondary engines
- Respects cost sensitivity preferences
- Returns routing rationale + cost estimate

### 3. Health Monitoring

Automated health checks for all engines:
- **PNC:** MCP tool health check
- **PNG:** CLI version check (`agy --version`)
- **PNO:** HTTP API check (`curl http://192.168.50.20:11434/api/tags`)
- **PNX:** CLI version check (`codex --version`)
- **PNK:** CLI version check (`opencode --version`)

Health status cached for 60 seconds to avoid excessive checks.

## Task Classification

Automatically detects task types:

| Task Type | Example Keywords | Routes To | Cost |
|-----------|------------------|-----------|------|
| **web_research** | research, search, find information, lookup | PNG | Free |
| **fast_classification** | classify, categorize, label, sentiment | PNO | Free |
| **structured_json** | json, extract, parse, structured | PNO | Free |
| **summarization** | summarize, condense, brief, tldr | PNO | Free |
| **multi_step_reasoning** | reason about, evaluate, compare | PNO | Free |
| **code_generation_simple** | implement, build, function | PNK | Paid |
| **code_generation_complex** | architecture, multi-file, system design | PNC | Paid |
| **adversarial_verification** | verify, audit, security, cato | PNX | Paid |
| **algorithm_execution** | algorithm mode, observe, ISC | PNC | Paid |

## Usage

### CLI

```bash
# Route a task
bun PAI/Tools/EngineRouter.ts route "research Cloudflare Workers security"
# Returns: { engine: "PNG", rationale: "...", cost_estimate: "Free" }

# Check health of all engines
bun PAI/Tools/EngineRouter.ts health-check

# Check specific engine health
bun PAI/Tools/EngineRouter.ts health-check PNO

# Get engine capabilities
bun PAI/Tools/EngineRouter.ts capabilities PNC

# List all engines
bun PAI/Tools/EngineRouter.ts list
```

### Programmatic (TypeScript)

```typescript
import { routeTask, getCapabilities } from "./EngineRouter";

// Route with health check + cost awareness
const result = await routeTask("classify these 50 log files");
console.log(result);
// {
//   engine: "PNO",
//   rationale: "Ollama zero-cost local inference...",
//   fallback: ["PNC"],
//   cost_estimate: "Free",
//   health_checked: true
// }

// Override cost sensitivity
const result2 = await routeTask("complex analysis", {
  costSensitivity: "free_only" // Force free engines only
});

// Skip health check for speed
const result3 = await routeTask("research topic", {
  checkHealth: false
});
```

## Cost Sensitivity Levels

| Level | Behavior | Use When |
|-------|----------|----------|
| **free_only** | Only route to PNO/PNG (zero-cost engines) | Budget-constrained, bulk operations |
| **free_first** | Prefer free, fallback to paid if needed | Normal operations |
| **balanced** | Balance cost vs quality | Default |
| **quality_first** | Prefer best engine regardless of cost | Critical tasks, final verification |

## Routing Rules

### Web Research → PNG (Antigravity)
- **Primary:** PNG (Gemini, free)
- **Fallback:** PNC
- **Rationale:** PNG specialized for web research with parallel search

### Fast Classification → PNO (Ollama)
- **Primary:** PNO (llama3.2:3b, free)
- **Fallback:** PNC
- **Rationale:** Local inference, zero API cost

### Structured JSON → PNO (Ollama)
- **Primary:** PNO (qwen2.5:7b, free)
- **Fallback:** PNC
- **Rationale:** qwen optimized for JSON extraction

### Summarization → PNO (Ollama)
- **Primary:** PNO (gemma2:9b, free)
- **Fallback:** PNC
- **Rationale:** Local summarization handles most cases

### Multi-Step Reasoning → PNO (Ollama)
- **Primary:** PNO (deepseek-r1:7b, free)
- **Fallback:** PNC
- **Rationale:** Local reasoning with escalation path

### Code Generation (Simple) → PNK (OpenCode)
- **Primary:** PNK
- **Fallback:** PNC, PNX
- **Rationale:** Provider-agnostic multi-file builder

### Code Generation (Complex) → PNC (Claude Code)
- **Primary:** PNC
- **Fallback:** PNK
- **Rationale:** Full Algorithm for architecture work

### Adversarial Verification → PNX (Codex)
- **Primary:** PNX
- **Fallback:** PNC
- **Rationale:** Cross-vendor audit via Cato

### Algorithm Execution → PNC (Claude Code)
- **Primary:** PNC only
- **Fallback:** None
- **Rationale:** Only PNC has full Algorithm capability

## Failover Behavior

1. **Primary Healthy:** Use primary engine
2. **Primary Degraded/Offline:** Try first fallback
3. **All Fallbacks Offline:** Attempt primary anyway with warning
4. **Cost Constraint:** Skip paid engines if `free_only` or budget exceeded

## Health Status

Engines can be in three states:

| Status | Meaning | Router Behavior |
|--------|---------|-----------------|
| **healthy** | All checks passing | Use for routing |
| **degraded** | Partial functionality | Skip, try fallback |
| **offline** | Unavailable | Skip, try fallback |

Health checks run automatically before routing (unless `checkHealth: false`).

Cache duration: 60 seconds (configurable via `HEALTH_CACHE_TTL_MS`).

## Integration Points

### Algorithm OBSERVE Phase

Before expensive operations:

```typescript
// In Algorithm OBSERVE
const route = await routeTask(userRequest);
if (route.engine !== "PNC") {
  // Present routing recommendation via mcp_Question
  // "This task can be handled by {route.engine} ({route.cost_estimate})"
}
```

### Budget Guard Hook

```typescript
// In BudgetTracker.hook.ts
const budget = checkBudgetConstraints();
if (!budget.can_use_paid) {
  // Force free-only routing
  const route = await routeTask(task, { costSensitivity: "free_only" });
}
```

### OmniPulse Dashboard

Real-time engine health displayed in Command Center:

```
Engine Health:
PNC ● healthy (2s ago)
PNG ● healthy (15s ago)
PNO ● healthy (1s ago)
PNX ○ offline (2h ago)
PNK ● healthy (30s ago)
```

## Testing

```bash
# Run automated test suite
cd PAI/Tools
bun test EngineRouter.test.ts

# Manual routing tests
bun EngineRouter.ts route "research AI safety"
bun EngineRouter.ts route "classify sentiment"
bun EngineRouter.ts route "build auth system"
bun EngineRouter.ts route "audit security"
```

## Performance

- **Classification:** <1ms (keyword matching)
- **Health Check (cached):** <1ms (file read)
- **Health Check (fresh):** 50-200ms (5 parallel checks)
- **Routing Decision:** <5ms total

## Future Enhancements

### Planned (v1.1)
1. **Budget Integration:** Automatic free-only routing when budget low
2. **Parallel Engine Execution:** Run task on multiple engines, synthesize
3. **Cost Tracking:** Record actual costs per engine per task
4. **Performance Metrics:** Track success rate, latency per engine
5. **Auto-Failover:** Automatic retry on engine timeout/error

### Considered (v2.0)
6. **ML-Based Classification:** Learn from routing decisions over time
7. **Task Complexity Scoring:** Estimate effort before routing
8. **Cross-Engine Agent Teams:** Coordinate agents from different engines
9. **Async Task Queue:** Submit background work to any engine
10. **Engine Load Balancing:** Distribute work based on current load

## Files

```
PAI/Tools/
├── engine-capability-matrix.json    # Capability + routing config
├── EngineRouter.ts                  # Smart router implementation
├── EngineRouter.test.ts             # Test suite
└── lib/
    └── pai-dir.ts                   # Path resolution

~/MEMORY/STATE.claude/
└── engine-health.json               # Cached health status

PAI/DOCUMENTATION/
└── ENGINE_ORCHESTRATION.md          # This file
```

## Configuration

Edit `engine-capability-matrix.json` to:
- Add new capabilities
- Adjust routing rules
- Change health check methods
- Update cost estimates
- Modify fallback chains

## Troubleshooting

### Engine Always Routes to PNC

**Cause:** Task classification not matching any specific pattern  
**Fix:** Add keywords to `patterns` array in `classifyTask()` function

### Health Check Failing

**Cause:** Command or regex pattern mismatch  
**Fix:** Test health check command manually:
```bash
bash -c "which agy && agy --version"
```
Adjust `expected_pattern` in matrix if needed.

### Free-Only Routing Using Paid Engine

**Cause:** Task type doesn't have a free primary route  
**Fix:** Add routing rule with free primary or adjust classification

### All Engines Offline

**Cause:** Health check timeout or network issue  
**Fix:** Check health cache file, run manual health check

## Related Documentation

- `INTERACTIVITY.md` — Cross-engine user prompts
- `budget-shared.ts` — Budget tracking system
- `CLAUDE.md` — PNC configuration
- `GEMINI.md` — PNG configuration
- `AGENTS.md` (per-engine) — Engine-specific rules

## Changelog

### 1.0.0 (2026-07-05)
- Initial implementation
- 5-engine capability matrix
- Smart task classification
- Health monitoring with failover
- Cost-aware routing
- Automated test suite
- CLI and programmatic interfaces
