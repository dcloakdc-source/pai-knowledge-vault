# PAI Engine Dashboard

**Version:** 1.0.0  
**Date:** 2026-07-05  
**Status:** Production Ready

## Overview

Real-time cross-engine orchestration dashboard displaying live status of all 5 PAI engines (PNC, PNG, PNO, PNX, PNK) with health monitoring, budget tracking, and task coordination.

## Features

- **Engine Status:** Health, current activity, elapsed time
- **Budget Monitoring:** Window/weekly usage, cost tracking, routing recommendations
- **Handoff Tracking:** Active/pending/failed cross-engine handoffs
- **Task Queue:** Queued/running/completed tasks
- **Live Updates:** Auto-refresh every 2 seconds
- **Multiple Modes:** TUI, JSON, snapshot

## Usage

### Live TUI Dashboard

```bash
bun PAI/Tools/EngineDashboard.ts
# Press Ctrl+C to exit
```

**Output:**
```
╔═══════════════════════════════════════════════════════════════╗
║PAI Orchestration Dashboard                                    ║
║3:33:18 PM                                                     ║
╠═══════════════════════════════════════════════════════════════╣
║ PNC │ ●  │ E3 BUILD (15min)                                   ║
║ PNG │ ●  │ RESEARCH                                           ║
║ PNO │ ●  │ READY                                              ║
║ PNX │ ○  │ OFFLINE                                            ║
║ PNK │ ●  │ 4-file refactor (8min)                             ║
╠═══════════════════════════════════════════════════════════════╣
║Budget: 73.4% ⚠️  (free_first)                                  ║
║Cost (7d): $2.3456                                             ║
╠═══════════════════════════════════════════════════════════════╣
║Active Handoffs: 2  │  Pending: 1                              ║
║Queued Tasks: 5  │  Running: 2                                 ║
║Failed Today: 1                                                ║
╚═══════════════════════════════════════════════════════════════╝

Refresh: 2s  │  Ctrl+C to exit
```

### Single Snapshot

```bash
bun PAI/Tools/EngineDashboard.ts --once
# Displays current state, then exits
```

### JSON Output

```bash
bun PAI/Tools/EngineDashboard.ts --json
```

**Returns:**
```json
{
  "timestamp": "2026-07-05T15:33:18.000Z",
  "engines": [
    {
      "engine": "PNC",
      "health": "healthy",
      "activity": "E3 BUILD",
      "elapsed": "15min",
      "last_seen": "2026-07-05T15:33:15.000Z"
    },
    {
      "engine": "PNG",
      "health": "healthy",
      "activity": "RESEARCH",
      "last_seen": "2026-07-05T15:32:10.000Z"
    },
    {
      "engine": "PNO",
      "health": "healthy",
      "activity": "READY"
    },
    {
      "engine": "PNX",
      "health": "offline",
      "activity": "IDLE",
      "last_seen": "2026-07-05T13:15:00.000Z"
    },
    {
      "engine": "PNK",
      "health": "healthy",
      "activity": "4-file refactor",
      "elapsed": "8min"
    }
  ],
  "budget": {
    "window_percent": 73.4,
    "weekly_percent": 68.2,
    "cost_7day": 2.3456,
    "recommended_routing": "free_first"
  },
  "handoffs": {
    "active": 2,
    "pending": 1,
    "failed": 1
  },
  "tasks": {
    "queued": 5,
    "running": 2,
    "completed_today": 12
  }
}
```

## Engine Status Indicators

### Health Icons

| Icon | Status | Meaning |
|------|--------|---------|
| ● (green) | `healthy` | All checks passing, ready for work |
| ◐ (yellow) | `degraded` | Partial functionality, use with caution |
| ○ (red) | `offline` | Unavailable, use fallback |

### Activity States

| Engine | States |
|--------|--------|
| **PNC** | IDLE, E1-E5 OBSERVE/THINK/PLAN/BUILD/VERIFY/LEARN |
| **PNG** | IDLE, RESEARCH, SEARCH |
| **PNO** | READY (stateless) |
| **PNX** | IDLE, VERIFY, CATO, SANDBOX |
| **PNK** | IDLE, PLAN, BUILD, {N}-file work |

### Budget Indicators

| Percent | Icon | Routing |
|---------|------|---------|
| 0-50% | ✓ (green) | quality_first |
| 50-70% | ✓ (green) | balanced |
| 70-90% | ⚠️ (yellow) | free_first |
| 90-100% | 🚨 (red) | free_only |

## Data Sources

### Engine Health
- **Source:** `~/MEMORY/STATE.claude/engine-health.json`
- **Updated by:** `EngineRouter.ts` health checks (60s cache)
- **Fallback:** Unknown status if cache missing

### PNC Activity
- **Source:** `~/MEMORY/STATE.claude/algorithm-phase.json`
- **Updated by:** Algorithm voice curls + phase transitions
- **Fallback:** IDLE if file missing

### Budget Status
- **Source:** `CrossEngineBudget.ts` checkBudget()
- **Integrates:** PNC token budget + cross-engine costs
- **Refresh:** Real-time (no cache)

### Handoffs
- **Source:** `RecordRouter.ts list` command
- **Tracks:** OmniPulse cross-engine handoffs
- **Status:** active/pending/failed

### Tasks
- **Source:** Task queue (future implementation)
- **Current:** Stub returning zeros
- **TODO:** Integrate with actual task queue system

## Integration Points

### OmniPulse Gateway

Dashboard can be served via OmniPulse at `http://localhost:31338/dashboard`:

```typescript
// In OmniPulse gateway
app.get("/dashboard", async (req, res) => {
  const result = execSync("bun PAI/Tools/EngineDashboard.ts --json");
  res.json(JSON.parse(result));
});
```

### Command Center

Legacy Command Center integration (deprecated, use OmniPulse):

```bash
# Command Center used to serve at :8766
# Now merged into OmniPulse :31338
```

### Tmux Status Line

Add engine status to tmux:

```bash
# In .tmux.conf
set -g status-right '#(bun PAI/Tools/EngineDashboard.ts --json | jq -r ".engines[] | select(.health!=\"healthy\") | .engine" | tr "\n" " ")'
```

### Monitoring/Alerting

Export metrics for Prometheus/Grafana:

```bash
# Cron job every 1 minute
* * * * * bun PAI/Tools/EngineDashboard.ts --json > /var/lib/prometheus/node_exporter/engine_status.prom
```

## Programmatic Usage

```typescript
import { getDashboardState, renderDashboard, renderJSON } from "./EngineDashboard";

// Get current state
const state = getDashboardState();

// Check if any engine is offline
const offline = state.engines.filter((e) => e.health === "offline");
if (offline.length > 0) {
  console.warn(`Engines offline: ${offline.map((e) => e.engine).join(", ")}`);
}

// Check budget status
if (state.budget.window_percent > 90) {
  console.error("Budget critical! Routing to free engines only");
}

// Render ASCII
console.log(renderDashboard(state));

// Render JSON
console.log(renderJSON(state));
```

## Future Enhancements

### Planned (v1.1)
1. **Real-time task queue** — Integrate with actual async task system
2. **PNG/PNX/PNK activity detection** — Parse engine-specific state files
3. **Historical trends** — Show usage over time
4. **Alert thresholds** — Configurable warnings (email/voice/ntfy)
5. **Web UI** — Browser-based dashboard with charts

### Considered (v2.0)
6. **Engine load distribution** — Show token usage per engine
7. **Cost forecasting** — Predict when budget limit hits
8. **Session attribution** — Link activities to sessions
9. **Performance metrics** — Latency, success rate, failover count
10. **Multi-user support** — Per-user dashboard views

## Files

```
PAI/Tools/
├── EngineDashboard.ts          # Dashboard implementation
├── EngineRouter.ts             # Health checks
├── CrossEngineBudget.ts        # Budget status
└── RecordRouter.ts             # Handoff tracking

~/MEMORY/STATE.claude/
├── engine-health.json          # Cached health status
├── algorithm-phase.json        # PNC activity state
├── engine-usage.jsonl          # Cross-engine usage
└── token-budget.jsonl          # PNC token usage

PAI/DOCUMENTATION/
├── ENGINE_DASHBOARD.md         # This file
├── ENGINE_ORCHESTRATION.md     # Router docs
└── CROSS_ENGINE_BUDGET.md      # Budget docs
```

## Troubleshooting

### Dashboard shows all engines IDLE but work is happening

**Cause:** Activity detection not implemented for PNG/PNX/PNK  
**Fix:** These engines need state files similar to `algorithm-phase.json`

### Health status shows "unknown"

**Cause:** Health cache expired or never populated  
**Fix:** Run `bun EngineRouter.ts health-check` to refresh

### Handoffs always show 0

**Cause:** No active OmniPulse handoffs  
**Fix:** This is normal if no cross-engine coordination is happening

### Budget shows 0%

**Cause:** Budget not configured (window_output_limit = 0)  
**Fix:** Run `bun BudgetReport.ts --set-window-limit <limit>`

## Related Documentation

- `ENGINE_ORCHESTRATION.md` — Engine routing system
- `CROSS_ENGINE_BUDGET.md` — Budget tracking
- `INTERACTIVITY.md` — User prompts

## Changelog

### 1.0.0 (2026-07-05)
- Initial dashboard implementation
- Engine health monitoring
- Budget status display
- Handoff tracking
- TUI, JSON, and snapshot modes
- Real-time updates (2s refresh)
- Programmatic API
