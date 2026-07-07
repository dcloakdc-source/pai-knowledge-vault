# Agent Loop Observability System

**Status**: Production-ready (2026-06-30)
**Research basis**: Cole Medin - Loop engineering replacing manual prompting

## Overview

The Agent Loop Observability System provides real-time monitoring of PAI's autonomous agent orchestration patterns. As loop engineering (24/7 autonomous orchestration) replaces manual prompting, this system gives visibility into:

- **Loop lifecycles** - Start, iterations, completion/failure
- **Token usage tracking** - Cost analysis per loop and iteration
- **Orchestrator→Worker patterns** - Dependency graphs showing agent communication
- **Performance metrics** - Success rates, throughput, bottlenecks
- **Real-time dashboard** - WebSocket-powered live monitoring

## Architecture

```
AgentLoopMonitor.ts           Core tracking & JSONL event log
  ↓
agent-loops.jsonl             Append-only event stream
agent-loops/                  Event broadcast directory
  ↓
AgentLoopDashboard/server.ts  WebSocket server
  ↓
AgentLoopDashboard/index.html Real-time web UI
```

### Design Principles

1. **Append-only JSONL** - Crash-safe, grep-able, no database required
2. **Event-driven** - Monitor emits events, dashboard subscribes
3. **Decoupled** - Monitor works without dashboard running
4. **Zero-dependency** - Built on Bun + native modules
5. **CLI-first** - Dashboard is visualization layer

## Usage

### Programmatic API

```typescript
import { logLoopStart, logIteration, logLoopEnd } from '~/.claude/PAI/Tools/AgentLoopMonitor';

// Start a loop
const loopId = await logLoopStart({
  orchestratorId: "main-orchestrator",
  pattern: "research-loop",
  metadata: { topic: "agent observability" }
});

// Log iterations
await logIteration(loopId, {
  iterationNum: 1,
  workerId: "researcher-123",
  status: "success",
  tokensUsed: 1500
});

await logIteration(loopId, {
  iterationNum: 2,
  workerId: "synthesizer-456",
  status: "success",
  tokensUsed: 2100
});

// End loop
await logLoopEnd(loopId, {
  status: "completed",
  totalIterations: 2,
  reason: "goal achieved"
});
```

### CLI Commands

```bash
# View active loops
bun ~/.claude/PAI/Tools/AgentLoopMonitor.ts status

# Get metrics for a specific loop
bun ~/.claude/PAI/Tools/AgentLoopMonitor.ts metrics <loopId>

# Show orchestrator→worker dependency graph
bun ~/.claude/PAI/Tools/AgentLoopMonitor.ts graph

# View recent event history
bun ~/.claude/PAI/Tools/AgentLoopMonitor.ts history [limit]

# Prune old completed loops
bun ~/.claude/PAI/Tools/AgentLoopMonitor.ts prune [days]
```

### Dashboard

Start the real-time dashboard:

```bash
bun ~/.claude/PAI/Tools/AgentLoopDashboard/server.ts
```

Then open: http://localhost:8765

**Features:**
- Live loop status with iteration counts
- Real-time event stream
- Token usage tracking
- Orchestrator→Worker dependency visualization
- Auto-reconnecting WebSocket (survives server restarts)

### Integration with Existing Systems

The monitor integrates with:

- **AgentLogger.ts** - Tracks agent activity phases
- **WorkLogger.ts** - Records work types and phases
- **Delegation skill** - Wave-based parallel execution tracking

To add loop tracking to an orchestrator:

```typescript
import { logLoopStart, logIteration, logLoopEnd } from '~/.claude/PAI/Tools/AgentLoopMonitor';

async function myOrchestrator() {
  const loopId = await logLoopStart({
    orchestratorId: "my-orchestrator",
    pattern: "custom-loop"
  });

  let iteration = 0;
  while (!goalAchieved) {
    iteration++;
    try {
      const result = await doWork();
      await logIteration(loopId, {
        iterationNum: iteration,
        status: "success",
        tokensUsed: result.tokensUsed
      });
    } catch (error) {
      await logIteration(loopId, {
        iterationNum: iteration,
        status: "failed",
        error: error.message
      });
      break;
    }
  }

  await logLoopEnd(loopId, {
    status: "completed",
    totalIterations: iteration
  });
}
```

## Event Schema

All events are JSONL with this structure:

```typescript
interface LoopEvent {
  timestamp: string;              // ISO 8601
  event: "loop_start" | "iteration" | "loop_end";
  loopId: string;                 // Unique loop identifier
  orchestratorId?: string;        // Orchestrator name (loop_start)
  pattern?: string;               // Loop pattern type (loop_start)
  iterationNum?: number;          // Iteration number (iteration)
  workerId?: string;              // Worker identifier (iteration)
  status?: string;                // "success" | "failed" | "timeout" | "completed"
  tokensUsed?: number;            // Token count for this iteration
  totalIterations?: number;       // Final count (loop_end)
  error?: string;                 // Error message if failed
  reason?: string;                // Completion reason (loop_end)
  metadata?: Record<string, any>; // Custom data
}
```

## Metrics

The system tracks:

- **totalIterations** - Number of iterations executed
- **successCount** - Successful iterations
- **failureCount** - Failed iterations
- **timeoutCount** - Timed-out iterations
- **successRate** - Percentage of successful iterations
- **totalTokens** - Cumulative token usage
- **avgTokensPerIteration** - Average token cost
- **duration** - Total loop runtime (ms)
- **throughput** - Iterations per minute

## Storage

All data lives in `~/MEMORY/STATE.claude/`:

```
agent-loops.jsonl          Main event log (append-only)
agent-loops/               Event broadcast directory (ephemeral)
  event-<timestamp>.json   Individual events for WebSocket broadcast
```

Events older than 30 days can be pruned:

```bash
bun ~/.claude/PAI/Tools/AgentLoopMonitor.ts prune 30
```

## Testing

Comprehensive test suite at:

```
~/.claude/PAI/Tools/__tests__/agent-loop-monitor/monitor.test.ts
```

Run tests:

```bash
cd ~/.claude/PAI/Tools
bun test __tests__/agent-loop-monitor/monitor.test.ts
```

## Future Enhancements

Potential additions:

1. **Cost tracking** - Integrate with Claude API pricing
2. **Alerting** - Notify on loop failures or anomalies
3. **Historical analysis** - Trend analysis across loop types
4. **Performance profiling** - Bottleneck identification
5. **Hook integration** - Auto-track all Algorithm runs

## References

- Cole Medin research: Loop engineering > manual prompting
- Delegation skill: `~/.claude/skills/Delegation/SKILL.md`
- AgentLogger: `~/.claude/PAI/Tools/AgentLogger.ts`
- WorkLogger: `~/.claude/PAI/Tools/WorkLogger.ts`
