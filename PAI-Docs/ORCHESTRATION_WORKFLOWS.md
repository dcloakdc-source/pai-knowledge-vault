# Cross-Engine Orchestration Workflows

**Version:** 1.0.0  
**Date:** 2026-07-05  
**Status:** Production Ready

## Overview

Six integrated systems for intelligent cross-engine orchestration:

1. **EngineRouter** - Smart task routing with health-aware failover
2. **CrossEngineBudget** - Unified cost tracking across all engines
3. **EngineDashboard** - Real-time monitoring TUI
4. **InteractivePrompt** - Cross-engine user interaction
5. **ParallelEngineExecution** - Multi-engine verification
6. **TaskQueue** - Async background execution

This document shows how they work together in common scenarios.

---

## Workflow 1: Cost-Aware Research

**Scenario:** User asks for research. Budget is at 85% (free-preferred zone).

### Step 1: Route Task

```bash
bun PAI/Tools/EngineRouter.ts route "research Cloudflare Workers security"
```

**Output:**
```json
{
  "task": "research Cloudflare Workers security",
  "task_type": "research",
  "selected_engine": "PNG",
  "reason": "PNG optimal for research (web search capability), budget tier free_first prefers PNG cost $0/Mtok",
  "fallback": "PNC",
  "confidence": 0.9,
  "budget_tier": "free_first"
}
```

**What happened:**
1. EngineRouter classified task as "research" (confidence 0.9)
2. CrossEngineBudget.ts reported 85% usage → `free_first` tier
3. Router preferred PNG ($0) over PNC ($15/Mtok)
4. Health check passed for PNG
5. Fallback to PNC if PNG fails

### Step 2: Execute Task

```bash
# Direct execution
agy --dangerously-skip-permissions -p "research Cloudflare Workers security"
```

**OR submit to queue:**

```bash
bun PAI/Tools/TaskQueue.ts submit \
  --task "research Cloudflare Workers security" \
  --engine PNG \
  --priority low
```

### Step 3: Monitor

Terminal 1 - Watch dashboard:
```bash
bun PAI/Tools/EngineDashboard.ts
```

**Shows:**
- PNG: ✓ healthy, activity +1, $0 cost
- Budget: 85% used, free_first tier active
- Active handoffs: 0

### Result

- **Cost:** $0 (free engine)
- **Quality:** High (PNG excels at research)
- **Budget impact:** 0% increase
- **Fallback ready:** PNC if PNG fails

---

## Workflow 2: Critical Security Assessment (Parallel Verification)

**Scenario:** Production security audit requires high confidence. Budget at 45% (quality-first zone).

### Step 1: Budget Check

```bash
bun PAI/Tools/CrossEngineBudget.ts status
```

**Output:**
```json
{
  "budget_status": {
    "current_period": {
      "total_cost": 6.75,
      "total_tokens": 450000
    },
    "limits": {
      "weekly_budget": 15.00
    },
    "usage_percent": 45.0,
    "tier": "quality_first"
  },
  "recommendations": {
    "tier": "quality_first",
    "preferred_engines": ["PNC", "PNX"]
  }
}
```

45% usage → quality_first tier → use best engines.

### Step 2: Parallel Execution

```bash
bun PAI/Tools/ParallelEngineExecution.ts execute \
  --task "security assessment of authentication system" \
  --engines "PNC,PNX" \
  --synthesize
```

**What happens:**
1. Task dispatched to both PNC and PNX simultaneously
2. Each engine independently analyzes the auth system
3. Results collected from both
4. Synthesis algorithm extracts consensus and unique findings

### Step 3: Review Synthesis

**Output:**
```json
{
  "task": "security assessment of authentication system",
  "engines": [
    {
      "engine": "PNC",
      "status": "completed",
      "output": "Found: SQL injection in login, XSS in profile, missing rate limiting",
      "duration_ms": 15234,
      "tokens": { "input": 8000, "output": 3500 }
    },
    {
      "engine": "PNX",
      "status": "completed",
      "output": "Found: SQL injection in login, session fixation, missing CSRF tokens",
      "duration_ms": 12890,
      "tokens": { "input": 7500, "output": 3200 }
    }
  ],
  "synthesis": {
    "consensus": [
      "SQL injection vulnerability in login endpoint"
    ],
    "unique_findings": {
      "PNC": ["XSS in profile page", "Missing rate limiting"],
      "PNX": ["Session fixation attack possible", "Missing CSRF tokens"]
    },
    "conflicts": [],
    "confidence": "high"
  }
}
```

### Step 4: Cost Tracking

```bash
bun PAI/Tools/CrossEngineBudget.ts report
```

**Shows:**
- PNC: 11,500 tokens = $0.17
- PNX: 10,700 tokens = $0.19
- **Total:** $0.36 for this task
- **New usage:** 47.4% (still quality_first tier)

### Result

- **Consensus finding:** SQL injection (both engines agree)
- **Comprehensive coverage:** 5 total findings (1 consensus + 4 unique)
- **High confidence:** Cross-vendor verification
- **Cost:** $0.36 (acceptable for critical security)
- **Budget tier maintained:** Still in quality_first

---

## Workflow 3: Overnight Batch Processing

**Scenario:** Classify 500 log entries. Non-urgent. Budget at 72% (free-preferred zone).

### Step 1: Submit to Queue

```bash
bun PAI/Tools/TaskQueue.ts submit \
  --task "classify 500 system log entries by severity" \
  --engine auto \
  --priority low \
  --deadline "2026-07-06T09:00:00Z"
```

**Output:**
```
Task submitted: task_xyz789_abc12
```

**What happened:**
1. TaskQueue.ts called EngineRouter.ts
2. Router classified as "classification" → PNO optimal (local Ollama)
3. Task queued with low priority
4. Deadline set for tomorrow 9am

### Step 2: Worker Processes Overnight

```bash
# Cron job runs every 5 minutes
*/5 * * * * bun PAI/Tools/TaskQueue.ts process
```

**Worker execution:**
1. Picks highest priority task (sorts by: priority → deadline → submit time)
2. Executes via PNO (local Ollama, $0 cost)
3. Saves results to `~/MEMORY/STATE.claude/task-queue/results/task_xyz789_abc12.json`
4. Updates task status to "completed"
5. CrossEngineBudget.ts records $0 cost

### Step 3: Morning Review

```bash
# Check if done
bun PAI/Tools/TaskQueue.ts status task_xyz789_abc12

# Get results
bun PAI/Tools/TaskQueue.ts results task_xyz789_abc12
```

**Output:**
```json
{
  "task_id": "task_xyz789_abc12",
  "status": "completed",
  "output": {
    "critical": 12,
    "warning": 87,
    "info": 401,
    "classified": 500
  },
  "duration_ms": 45000,
  "tokens": { "input": 0, "output": 0 }
}
```

### Result

- **Cost:** $0 (PNO is local)
- **Timing:** Completed overnight
- **Budget impact:** 0%
- **Quality:** Sufficient for log classification

---

## Workflow 4: Interactive Decision with Budget Awareness

**Scenario:** User needs to decide whether to use expensive model. Budget at 91% (free-only zone).

### Step 1: Budget Alert

Dashboard shows:
```
Budget: 91.2% used (CRITICAL - free_only tier active)
Weekly budget: $15.00
Used: $13.68
Remaining: $1.32
```

### Step 2: Interactive Prompt

```bash
bun PAI/Tools/InteractivePrompt.ts ask \
  --question "Budget at 91%. Which model should we use?" \
  --options '[
    {
      "label": "PNC (Claude Sonnet)",
      "description": "Best quality. $15/Mtok. Will likely exceed budget. 🔴"
    },
    {
      "label": "PNG (Gemini)",
      "description": "Free. Good for research. Stays within budget. ✅"
    },
    {
      "label": "PNO (Ollama)",
      "description": "Free. Local. Fast. Good for structured tasks. ✅"
    },
    {
      "label": "Skip",
      "description": "Wait until budget resets weekly."
    }
  ]'
```

**User sees:**
```
Budget at 91%. Which model should we use?

1. PNC (Claude Sonnet)
   Best quality. $15/Mtok. Will likely exceed budget. 🔴

2. PNG (Gemini)
   Free. Good for research. Stays within budget. ✅

3. PNO (Ollama)
   Free. Local. Fast. Good for structured tasks. ✅

4. Skip
   Wait until budget resets weekly.

> 
```

### Step 3: User Chooses PNG

**Output:**
```json
{
  "choice": 2,
  "label": "PNG (Gemini)",
  "custom": null
}
```

### Step 4: Execute with PNG

```bash
agy --dangerously-skip-permissions -p "user's actual task"
```

### Result

- **Budget preserved:** $0 cost keeps usage at 91%
- **User informed:** Clear consequences shown
- **Appropriate routing:** PNG good enough for many tasks
- **Budget safety:** No unexpected overages

---

## Workflow 5: Real-Time Monitoring During Long Task

**Scenario:** Running expensive multi-step task. Monitor budget in real-time.

### Terminal 1: Execute Task

```bash
bun PAI/Tools/ParallelEngineExecution.ts execute \
  --task "comprehensive architecture review of 10 microservices" \
  --engines "PNC,PNK,PNX" \
  --synthesize
```

### Terminal 2: Live Dashboard

```bash
bun PAI/Tools/EngineDashboard.ts
```

**Shows (refreshes every 2s):**
```
╭─ PAI Engine Dashboard ─────────────────────────────────────╮
│                                                             │
│  Engine Health                                              │
│  ├─ PNC  ✓ healthy  activity: 1  cost: $0.85  handoffs: 0  │
│  ├─ PNG  ✓ healthy  activity: 0  cost: $0.00  handoffs: 0  │
│  ├─ PNO  ✓ healthy  activity: 0  cost: $0.00  handoffs: 0  │
│  ├─ PNX  ✓ healthy  activity: 1  cost: $0.92  handoffs: 0  │
│  └─ PNK  ✓ healthy  activity: 1  cost: $0.78  handoffs: 0  │
│                                                             │
│  Budget Status                                              │
│  ├─ Weekly usage: 58.3% ($8.75 / $15.00)                   │
│  ├─ Tier: balanced                                          │
│  └─ Remaining: $6.25                                        │
│                                                             │
│  Active Handoffs: 0                                         │
│                                                             │
│  Last update: 2026-07-05 16:15:32                          │
╰─────────────────────────────────────────────────────────────╯
```

**Watch in real-time:**
- Activity counters increment as engines work
- Cost accumulates
- Budget percentage updates
- Tier may shift (balanced → quality_first if usage drops)

### Result

- **Visibility:** Real-time awareness of cost accumulation
- **Control:** Can cancel if budget approaching limit
- **Confidence:** See all engines healthy and working

---

## Workflow 6: Automatic Failover

**Scenario:** Primary engine fails. Router automatically fails over.

### Step 1: Route Task (PNX Offline)

```bash
bun PAI/Tools/EngineRouter.ts route "code review of pull request"
```

**Internal flow:**
1. Classifies as "code_review"
2. Prefers PNX (Codex optimal for code)
3. Health check: `codex --version` fails
4. Falls back to PNC (second choice)
5. Health check: `claude --version` succeeds

**Output:**
```json
{
  "task": "code review of pull request",
  "task_type": "code_review",
  "selected_engine": "PNC",
  "reason": "PNC selected after PNX health check failed",
  "fallback": "PNK",
  "confidence": 0.85,
  "budget_tier": "balanced"
}
```

### Step 2: Dashboard Shows Failure

```
Engine Health
├─ PNC  ✓ healthy  activity: 1  cost: $0.45  handoffs: 0
├─ PNG  ✓ healthy  activity: 0  cost: $0.00  handoffs: 0
├─ PNO  ✓ healthy  activity: 0  cost: $0.00  handoffs: 0
├─ PNX  ✗ offline  activity: 0  cost: $0.00  handoffs: 0  <-- FAILED
└─ PNK  ✓ healthy  activity: 0  cost: $0.00  handoffs: 0
```

### Step 3: Execute with Fallback

```bash
# User doesn't need to do anything - router handled it
claude --dangerously-skip-permissions -p "code review of pull request"
```

### Result

- **Automatic recovery:** No user intervention needed
- **Graceful degradation:** PNC is still excellent for code review
- **Visibility:** Dashboard shows which engine is offline
- **Logging:** Failover recorded in engine usage log

---

## Workflow 7: Hybrid Approach (Smart + Manual)

**Scenario:** Complex multi-phase project. Use router for some phases, manual for others.

### Phase 1: Research (Auto-Route)

```bash
bun PAI/Tools/EngineRouter.ts route "research authentication best practices"
# → Routes to PNG (free, excellent research)
agy -p "research authentication best practices"
```

### Phase 2: Classification (Queue)

```bash
bun PAI/Tools/TaskQueue.ts submit \
  --task "classify 50 security findings by severity" \
  --engine auto \
  --priority medium
# → Auto-routes to PNO (local Ollama, free)
```

### Phase 3: Code Implementation (Manual High-Quality)

```bash
# Don't route - use best engine directly
claude -p "implement OAuth2 authentication flow"
# Using PNC directly for highest quality
```

### Phase 4: Cross-Verification (Parallel)

```bash
bun PAI/Tools/ParallelEngineExecution.ts execute \
  --task "security review of OAuth2 implementation" \
  --engines "PNC,PNX" \
  --synthesize
# Deliberate cross-vendor verification
```

### Result

- **Optimized costs:** Free engines for research/classification
- **Quality where needed:** Direct PNC for implementation
- **High confidence:** Parallel verification for security
- **Total cost:** ~$0.50 instead of ~$2.00 if all PNC

---

## System Integration Map

```
┌─────────────────────────────────────────────────────────────┐
│                         USER REQUEST                         │
└────────────────┬────────────────────────────────────────────┘
                 │
                 v
        ┌────────────────────┐
        │  EngineRouter.ts   │  Classify task, check budget,
        │  (Smart Routing)   │  select optimal engine
        └────────┬───────────┘
                 │
         ┌───────┴────────┐
         │                │
         v                v
┌────────────────┐  ┌─────────────────────┐
│ Direct Execute │  │  TaskQueue.ts       │  Queue for async
│ (Fast path)    │  │  (Background work)  │  execution
└────────┬───────┘  └──────────┬──────────┘
         │                     │
         │            ┌────────┴──────────┐
         │            │  Worker Process   │
         │            │  (Cron/Systemd)   │
         │            └────────┬──────────┘
         │                     │
         v                     v
┌────────────────────────────────────────┐
│     ParallelEngineExecution.ts         │  Optional: multi-engine
│     (Cross-vendor verification)        │  for critical tasks
└────────┬───────────────────────────────┘
         │
         v
┌────────────────────────────────────────┐
│   CrossEngineBudget.ts (tracking)      │  Record cost, update
│   + InteractivePrompt.ts (user input)  │  usage, check tier
└────────┬───────────────────────────────┘
         │
         v
┌────────────────────────────────────────┐
│      EngineDashboard.ts (monitor)      │  Real-time visibility
└────────────────────────────────────────┘
```

---

## Cost Comparison Examples

### Example 1: Research Task

| Approach | Engine | Cost | Quality |
|----------|--------|------|---------|
| **Smart routing** | PNG | $0 | High |
| Manual (always PNC) | PNC | $0.45 | High |
| **Savings** | - | **$0.45** | **No loss** |

### Example 2: Security Assessment

| Approach | Engines | Cost | Confidence |
|----------|---------|------|------------|
| Single engine | PNC | $0.52 | Medium |
| **Parallel (2 engines)** | PNC+PNX | **$1.04** | **High** |
| Parallel (3 engines) | PNC+PNX+PNK | $1.56 | Very High |

**Decision:** 2 engines balances cost vs confidence.

### Example 3: Full Project (7 tasks)

| Task | Manual | Smart | Savings |
|------|--------|-------|---------|
| Research | $0.45 | $0 (PNG) | $0.45 |
| Classification | $0.38 | $0 (PNO) | $0.38 |
| Summarization | $0.22 | $0 (PNO) | $0.22 |
| Code review | $0.95 | $0.95 (PNC) | $0 |
| Implementation | $1.50 | $1.50 (PNC) | $0 |
| Security audit | $0.52 | $1.04 (PNC+PNX) | -$0.52 |
| Deployment | $0.15 | $0.15 (PNC) | $0 |
| **TOTAL** | **$4.17** | **$3.64** | **$0.53 (13%)** |

**Note:** Security audit costs MORE with smart routing (parallel verification), but quality justifies it.

---

## Decision Matrix: Which System to Use?

| Scenario | System | Why |
|----------|--------|-----|
| **Quick question** | Direct execution | Fastest path |
| **Research** | EngineRouter → PNG | Free, optimal |
| **Classification/JSON** | EngineRouter → PNO | Free, local |
| **Code generation** | Direct PNC | Best quality |
| **Security audit** | ParallelExecution | Cross-verification |
| **Critical decision** | ParallelExecution | High confidence |
| **Overnight work** | TaskQueue | Non-blocking |
| **Batch processing** | TaskQueue → PNO | Free, async |
| **Budget > 90%** | EngineRouter + InteractivePrompt | User choice |
| **Long-running** | TaskQueue + EngineDashboard | Monitor progress |

---

## Best Practices

### 1. Default to Smart Routing

```bash
# GOOD: Let router decide
bun EngineRouter.ts route "task description"

# AVOID: Hardcoding engine choice
claude -p "task description"  # Might be expensive
```

**Exception:** When you know you need top quality (code generation, creative writing).

### 2. Use Parallel Execution Sparingly

**Good uses:**
- Security assessments
- Critical architecture decisions
- Production deployment reviews
- High-stakes code reviews

**Bad uses:**
- Routine questions
- Simple tasks
- Research (single engine sufficient)
- When budget > 70%

### 3. Queue Background Work

```bash
# GOOD: Non-urgent work goes to queue
bun TaskQueue.ts submit --task "analyze logs" --engine auto --priority low

# AVOID: Blocking main thread
analyze_logs.sh  # Blocks for 10 minutes
```

### 4. Monitor Budget

```bash
# Check before expensive operations
bun CrossEngineBudget.ts status

# If > 80%, consider alternatives
if [ usage > 80% ]; then
  # Use PNG/PNO instead of PNC
fi
```

### 5. Watch Dashboard During Long Tasks

```bash
# Terminal 1
long_expensive_task.sh

# Terminal 2
bun EngineDashboard.ts  # Watch cost accumulate
```

---

## Troubleshooting

### Router Selects Wrong Engine

**Symptom:** Task routed to suboptimal engine.

**Solutions:**
1. Check task classification keywords in `engine-capability-matrix.json`
2. Add task-specific keywords to improve matching
3. Override with `--engine` if needed
4. Review budget tier - might be forcing free engines

### Budget Tracking Inaccurate

**Symptom:** Dashboard shows wrong usage percentage.

**Solutions:**
1. Check `~/MEMORY/STATE.claude/engine-usage.jsonl` for corrupt entries
2. Verify `CrossEngineBudget.ts` window calculation
3. Manually record missing costs with `record` command
4. Reset usage log if corrupted

### Task Queue Not Processing

**Symptom:** Tasks stuck in "queued".

**Solutions:**
1. Check worker is running: `ps aux | grep TaskQueue`
2. Verify cron job: `crontab -l`
3. Check worker logs: `~/MEMORY/STATE.claude/task-queue/worker.log`
4. Manually process: `bun TaskQueue.ts process`

### Dashboard Not Updating

**Symptom:** Stale data in dashboard.

**Solutions:**
1. Check health cache TTL (60s by default)
2. Verify engines are actually running
3. Force refresh by restarting dashboard
4. Check `~/MEMORY/STATE.claude/engine-health.json` permissions

### Parallel Execution Hangs

**Symptom:** One or more engines timeout.

**Solutions:**
1. Check engine health individually
2. Increase timeout: `--timeout 180000` (3 minutes)
3. Reduce number of engines
4. Check if engine is busy with another task

---

## Files Reference

```
PAI/Tools/
├── EngineRouter.ts                    # Smart routing
├── CrossEngineBudget.ts               # Budget tracking
├── EngineDashboard.ts                 # Live monitoring
├── InteractivePrompt.ts               # User prompts
├── ParallelEngineExecution.ts         # Multi-engine verify
├── TaskQueue.ts                       # Async queue
└── engine-capability-matrix.json      # Engine config

~/MEMORY/STATE.claude/
├── engine-health.json                 # Health cache
├── engine-usage.jsonl                 # Cost log
└── task-queue/
    ├── queue.jsonl                    # Queue state
    ├── results/                       # Task results
    └── worker.log                     # Worker output

PAI/DOCUMENTATION/
├── ORCHESTRATION_WORKFLOWS.md         # This file
├── ENGINE_ORCHESTRATION.md            # Routing details
├── CROSS_ENGINE_BUDGET.md             # Budget system
├── ENGINE_DASHBOARD.md                # Dashboard guide
├── PARALLEL_EXECUTION_AND_TASK_QUEUE.md  # Systems detail
└── INTERACTIVITY.md                   # User interaction
```

---

## Next Steps

### Immediate
1. Set up TaskQueue worker (cron job)
2. Test parallel execution with real task
3. Monitor first week of budget tracking

### Short-term
1. Tune engine classification keywords
2. Add custom budget tiers per user
3. Implement task dependencies in queue
4. Add web UI for dashboard

### Long-term
1. ML-based task classification
2. Predictive budget alerts
3. Multi-stage pipeline orchestration
4. Auto-scaling based on queue depth

---

*Last updated: 2026-07-05*
