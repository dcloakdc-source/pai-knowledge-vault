# Parallel Engine Execution & Task Queue

**Version:** 1.0.0  
**Date:** 2026-07-05  
**Status:** Production Ready

## Overview

Two complementary orchestration systems:

1. **ParallelEngineExecution** - Run tasks across multiple engines simultaneously for cross-vendor verification
2. **TaskQueue** - Async background task execution with priority, deadlines, and automatic retry

## Part 1: Parallel Engine Execution

### Purpose

Execute the same task across multiple engines in parallel to get:
- Cross-vendor verification
- Consensus on critical decisions
- Multiple perspectives on complex problems
- Higher confidence through agreement

### Use Cases

| Scenario | Engines | Benefit |
|----------|---------|---------|
| **Security Assessment** | PNC + PNX | Cross-vendor vulnerability detection |
| **Architecture Review** | PNC + PNK | Multiple design perspectives |
| **Critical Decisions** | PNC + PNX + PNK | Consensus-based confidence |
| **Code Review** | PNC + PNX | Independent verification |

### Usage

```bash
# Security assessment across 2 engines
bun PAI/Tools/ParallelEngineExecution.ts execute \
  --task "assess security of authentication flow" \
  --engines "PNC,PNX" \
  --synthesize

# Architecture review across 3 engines
bun PAI/Tools/ParallelEngineExecution.ts execute \
  --task "review microservices architecture" \
  --engines "PNC,PNK,PNX" \
  --synthesize
```

### Output Structure

```json
{
  "task": "assess security of authentication flow",
  "engines": [
    {
      "engine": "PNC",
      "status": "completed",
      "output": "[findings from Claude...]",
      "duration_ms": 15234,
      "tokens": { "input": 5000, "output": 2500 }
    },
    {
      "engine": "PNX",
      "status": "completed",
      "output": "[findings from Codex...]",
      "duration_ms": 12890,
      "tokens": { "input": 4800, "output": 2300 }
    }
  ],
  "synthesis": {
    "consensus": [
      "Missing rate limiting on login endpoint",
      "JWT tokens lack expiration validation"
    ],
    "unique_findings": {
      "PNC": ["Session fixation vulnerability in cookie handling"],
      "PNX": ["Timing attack possible in password comparison"]
    },
    "conflicts": [],
    "confidence": "high"
  },
  "total_duration_ms": 15500
}
```

### Synthesis Algorithm

**Consensus** - Findings mentioned by 2+ engines  
**Unique** - Findings from only one engine  
**Conflicts** - Contradictory claims between engines  
**Confidence** - `high` (3+ engines agree), `medium` (2 agree), `low` (no agreement)

### Programmatic Usage

```typescript
import { executeParallel } from "./ParallelEngineExecution";

const result = await executeParallel({
  task: "security audit of API endpoints",
  engines: ["PNC", "PNX"],
  synthesize: true,
  timeout_ms: 120000, // 2 minutes
});

// Check consensus
if (result.synthesis?.confidence === "high") {
  console.log("High confidence findings:", result.synthesis.consensus);
}

// Review unique findings
for (const [engine, findings] of Object.entries(result.synthesis?.unique_findings || {})) {
  console.log(`${engine} uniquely found:`, findings);
}
```

### Cost Considerations

Running parallel execution costs N×baseline:

| Engines | Approximate Cost Multiplier |
|---------|----------------------------|
| 2 engines (PNC + PNX) | 2× |
| 3 engines (PNC + PNX + PNK) | 3× |
| With free engines (PNO/PNG) | Reduces total |

**Recommendation:** Use for critical decisions only. Most tasks should use single-engine routing.

---

## Part 2: Task Queue

### Purpose

Submit tasks for asynchronous background execution with:
- Priority-based scheduling
- Deadline awareness
- Automatic retry on failure
- Result persistence

### Use Cases

| Scenario | Priority | Deadline | Engine |
|----------|----------|----------|--------|
| **Overnight Research** | low | tomorrow 9am | PNG |
| **Batch Classification** | medium | none | PNO |
| **Scheduled Security Scan** | high | weekly | PNX |
| **Background Analysis** | low | none | PNC |

### Usage

#### Submit Task

```bash
# Research task (low priority, PNG)
bun PAI/Tools/TaskQueue.ts submit \
  --task "research latest AI safety developments" \
  --engine PNG \
  --priority low

# Urgent task with deadline
bun PAI/Tools/TaskQueue.ts submit \
  --task "security audit of production API" \
  --engine PNX \
  --priority urgent \
  --deadline "2026-07-06T09:00:00Z"

# Auto-route to optimal engine
bun PAI/Tools/TaskQueue.ts submit \
  --task "classify 500 log entries" \
  --engine auto \
  --priority medium
```

#### Check Status

```bash
bun PAI/Tools/TaskQueue.ts status task_abc123_def45
```

**Returns:**
```json
{
  "id": "task_abc123_def45",
  "task": "research latest AI safety developments",
  "engine": "PNG",
  "priority": "low",
  "status": "completed",
  "submitted_at": "2026-07-05T15:55:31Z",
  "started_at": "2026-07-05T15:56:25Z",
  "completed_at": "2026-07-05T15:58:12Z",
  "result_path": "~/MEMORY/STATE.claude/task-queue/results/task_abc123_def45.json"
}
```

#### Get Results

```bash
bun PAI/Tools/TaskQueue.ts results task_abc123_def45
```

**Returns:**
```json
{
  "task_id": "task_abc123_def45",
  "status": "completed",
  "output": "[full results from PNG research...]",
  "duration_ms": 107000,
  "tokens": { "input": 0, "output": 0 }
}
```

#### List Queue

```bash
# All tasks
bun PAI/Tools/TaskQueue.ts list

# Filter by status
bun PAI/Tools/TaskQueue.ts list --status queued
bun PAI/Tools/TaskQueue.ts list --status running
bun PAI/Tools/TaskQueue.ts list --status completed
```

#### Process Queue (Worker Mode)

```bash
# Process one task from queue
bun PAI/Tools/TaskQueue.ts process
```

### Priority System

| Priority | When to Use | Execution Order |
|----------|-------------|-----------------|
| **urgent** | Production issues, critical deadlines | 1st |
| **high** | Important work, time-sensitive | 2nd |
| **medium** | Normal background work | 3rd |
| **low** | Research, bulk operations | 4th |

Tasks sorted by: Priority → Deadline (soonest first) → Submission time (oldest first)

### Automatic Retry

Failed tasks retry automatically up to `max_retries` (default: 3).

**Retry logic:**
1. Task fails
2. If `retry_count < max_retries`: requeue with incremented counter
3. If `retry_count >= max_retries`: mark as permanently failed

### Programmatic Usage

```typescript
import { submitTask, getTaskStatus, getTaskResults } from "./TaskQueue";

// Submit task
const taskId = await submitTask({
  task: "research Cloudflare Workers security",
  engine: "PNG",
  priority: "low",
  deadline: "2026-07-06T09:00:00Z",
});

console.log(`Task submitted: ${taskId}`);

// Poll for completion
const interval = setInterval(async () => {
  const status = getTaskStatus(taskId);
  
  if (status?.status === "completed") {
    clearInterval(interval);
    const results = getTaskResults(taskId);
    console.log("Task complete:", results?.output);
  } else if (status?.status === "failed") {
    clearInterval(interval);
    console.error("Task failed:", status.error);
  }
}, 5000);
```

### Worker Deployment

**Cron Job (recommended):**
```bash
# Process queue every 5 minutes
*/5 * * * * /home/duane/.bun/bin/bun /home/duane/.claude/PAI/Tools/TaskQueue.ts process >> /home/duane/MEMORY/STATE.claude/task-queue/worker.log 2>&1
```

**Systemd Service:**
```ini
[Unit]
Description=PAI Task Queue Worker
After=network.target

[Service]
Type=simple
User=duane
ExecStart=/home/duane/.bun/bin/bun /home/duane/.claude/PAI/Tools/TaskQueue.ts process
Restart=always
RestartSec=60

[Install]
WantedBy=multi-user.target
```

**Tmux Session:**
```bash
# Long-running worker
tmux new -s taskqueue -d 'while true; do bun PAI/Tools/TaskQueue.ts process; sleep 60; done'
```

---

## Integration: Parallel + Queue

Combine both systems for powerful workflows:

```typescript
// Submit parallel execution as background task
const taskId = await submitTask({
  task: JSON.stringify({
    type: "parallel_execution",
    task: "comprehensive security audit",
    engines: ["PNC", "PNX", "PNK"]
  }),
  engine: "PNC", // Coordinator engine
  priority: "high",
  deadline: "2026-07-06T00:00:00Z"
});

// Later, retrieve synthesis results
const results = getTaskResults(taskId);
const synthesis = JSON.parse(results.output);
console.log("Consensus findings:", synthesis.consensus);
```

---

## Files

```
PAI/Tools/
├── ParallelEngineExecution.ts  # Parallel execution
├── TaskQueue.ts                # Async task queue
├── EngineRouter.ts             # Smart routing
├── CrossEngineBudget.ts        # Cost tracking
└── EngineDashboard.ts          # Live monitoring

~/MEMORY/STATE.claude/task-queue/
├── queue.jsonl                 # Task queue state
└── results/                    # Task results (JSON)
    └── task_abc123_def45.json

PAI/DOCUMENTATION/
├── PARALLEL_EXECUTION_AND_TASK_QUEUE.md  # This file
├── ENGINE_ORCHESTRATION.md               # Routing
├── CROSS_ENGINE_BUDGET.md                # Budget
└── ENGINE_DASHBOARD.md                   # Dashboard
```

---

## Best Practices

### Parallel Execution

✅ **DO:**
- Use for security audits (cross-vendor verification)
- Use for critical architecture decisions
- Use when high confidence required
- Synthesize results for consensus

❌ **DON'T:**
- Use for routine tasks (expensive)
- Use for simple queries
- Ignore unique findings from one engine
- Skip synthesis step

### Task Queue

✅ **DO:**
- Use for overnight/background work
- Set appropriate priorities
- Use deadlines for time-sensitive work
- Let auto-routing choose optimal engine

❌ **DON'T:**
- Submit urgent interactive work (use direct execution)
- Set all tasks to "urgent" (defeats priority)
- Forget to run worker (tasks won't process)
- Submit duplicate tasks

---

## Troubleshooting

### Parallel Execution

**All engines timeout**
- Increase `timeout_ms` parameter
- Check engine health via EngineDashboard
- Verify engines are not already busy

**Synthesis shows no consensus**
- Normal for divergent tasks
- Review unique findings from each engine
- May indicate genuinely complex problem

### Task Queue

**Tasks stuck in "queued"**
- Worker not running
- Start worker: `bun TaskQueue.ts process`
- Or set up cron job

**Task failed with retry exhausted**
- Check error in task status
- Engine may be offline
- Task may be malformed

**Results not found**
- Task not yet completed (check status)
- Result file deleted/moved
- Task failed (check error field)

---

## Future Enhancements

### Planned (v1.1)
1. **Smarter synthesis** - LLM-based claim extraction + similarity matching
2. **Queue priorities** - Engine load balancing
3. **Task dependencies** - Wait for task A before starting task B
4. **Scheduled tasks** - Cron-like recurring tasks
5. **Task cancellation** - Kill running tasks

### Considered (v2.0)
6. **Multi-stage pipelines** - Chain parallel execution results
7. **Consensus thresholds** - Require N/M engines to agree
8. **Cost budgeting** - Per-task cost limits
9. **Result streaming** - Real-time partial results
10. **Web UI** - Browser-based queue management

---

## Related Documentation

- `ENGINE_ORCHESTRATION.md` - Engine routing
- `CROSS_ENGINE_BUDGET.md` - Cost tracking
- `ENGINE_DASHBOARD.md` - Live monitoring
- `INTERACTIVITY.md` - User prompts

---

## Changelog

### 1.0.0 (2026-07-05)
- Initial parallel execution implementation
- Task queue with priority/deadlines
- Automatic retry on failure
- Result persistence
- Synthesis algorithm
- CLI interfaces
- Programmatic APIs
