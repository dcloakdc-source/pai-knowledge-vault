# PAI Unified CLI - Simple Usage

**Version:** 1.0.0  
**Command:** `pai`  
**Status:** Production Ready

## No, You Don't Call EngineRouter Every Time

Instead, use the **`pai`** command - it does everything automatically.

---

## The Simple Way

### Just use `pai`:

```bash
# Research (auto-routes to PNG, free)
pai "research AI safety papers"

# Classification (auto-routes to PNO, free)
pai "classify this log entry as error or warning"

# Code generation (auto-routes to PNC, best quality)
pai "implement REST API endpoint with validation"
```

**That's it.** The system:
1. ✅ Automatically classifies your task
2. ✅ Selects optimal engine (cost + quality + health)
3. ✅ Executes on that engine
4. ✅ Shows you what it chose and why

---

## Common Commands

### 1. Normal Use (99% of the time)

```bash
pai "your task here"
```

**What happens:**
- Router analyzes task type
- Checks budget tier
- Selects optimal engine
- Executes immediately
- You see the result

**Examples:**
```bash
pai "research latest Claude features"
# → Routes to PNG (free)
# → Executes: agy -p "research latest Claude features"

pai "classify 10 support tickets by urgency"
# → Routes to PNO (free, local)
# → Executes: ollama run gemma2:9b "classify..."

pai "write Python function to parse CSV"
# → Routes to PNC (best for code)
# → Executes: claude -p "write Python function..."
```

---

### 2. Check Status (morning routine)

```bash
pai --status
```

**Shows:**
- Budget usage (%)
- Cost last 7 days
- Task queue status
- Engine health

**Example output:**
```
=== PAI System Status ===

Budget:
  Weekly usage: 16.5%
  Cost (7d): $0.0735
  Tier: quality_first
  
Task Queue:
  Queued: 0
  Running: 0
  
Engine Health:
  PNC ✓  PNG ✓  PNO ✓  PNX ✓  PNK ✓
```

---

### 3. Background Queue (overnight work)

```bash
pai --queue "analyze 500 log files"
```

**What happens:**
- Submits to background queue
- Router auto-selects engine (probably PNO - free)
- Worker processes when available
- Check results later

**With options:**
```bash
# High priority, deadline tomorrow 9am
pai --queue \
  --priority high \
  --deadline "2026-07-06T09:00:00Z" \
  "security scan of production"

# Check results
pai --status  # Shows queue status
```

---

### 4. Parallel Execution (critical work)

```bash
pai --parallel "security audit of authentication"
```

**What happens:**
- Runs on 2+ engines simultaneously (default: PNC + PNX)
- Synthesizes results (consensus + unique findings)
- Costs more but gives high confidence

**Custom engines:**
```bash
pai --parallel --engine "PNC,PNX,PNK" "architecture review"
# → 3-engine verification
```

---

### 5. Force Specific Engine (rare)

```bash
pai --engine PNC "your task"
```

**When to use:**
- You know you need highest quality (PNC)
- Testing specific engine
- Router would choose wrong one

**Example:**
```bash
# Force expensive engine despite budget
pai --engine PNC "critical production bug analysis"
```

---

### 6. Dashboard (watch long tasks)

```bash
pai --dashboard
```

**Shows:**
- Real-time engine status
- Budget percentage
- Queue depth
- Updates every 2 seconds

**Use when:**
- Running expensive parallel execution
- Want to see costs accumulate
- Monitoring overnight queue processing

---

## Your New Workflow

### Morning (10 seconds)

```bash
pai --status
```

See budget, queue, health. That's it.

---

### Every Task (just pai)

```bash
pai "your task"
```

No routing. No engine selection. No manual work.

---

### Before Bed (5 seconds)

```bash
pai --queue "batch work for overnight"
```

Wake up to results.

---

### Critical Security (30 seconds)

```bash
pai --parallel "security assessment"
```

Cross-vendor verification.

---

## When to Use What

| Scenario | Command | Why |
|----------|---------|-----|
| **Any normal task** | `pai "task"` | Auto-routes, optimal cost+quality |
| **Morning check** | `pai --status` | See budget/queue/health |
| **Overnight work** | `pai --queue "task"` | Non-blocking, wake to results |
| **Security audit** | `pai --parallel "task"` | Cross-vendor verification |
| **Force quality** | `pai --engine PNC "task"` | Override router |
| **Watch costs** | `pai --dashboard` | Real-time monitoring |

---

## What Happens Behind the Scenes

```
You: pai "research AI safety"
     ↓
[1] EngineRouter classifies task
     → Type: research
     → Confidence: 0.9
     ↓
[2] Check budget tier
     → 16.5% used
     → Tier: quality_first
     ↓
[3] Select optimal engine
     → PNG: $0, excellent for research
     → PNC fallback if PNG down
     ↓
[4] Health check
     → PNG: ✓ healthy
     ↓
[5] Execute
     → agy --dangerously-skip-permissions -p "research AI safety"
     ↓
[6] Show results
     → [research output from PNG]
```

**You see:**
```
🧭 Routing task...

✓ Selected: PNG (Free)
  Reason: PNG specialized for web research

▶ Executing: agy --dangerously-skip-permissions -p "research AI safety"

────────────────────────────────────────────────────────────
[results...]
────────────────────────────────────────────────────────────
```

---

## Advanced: Manual Tools (when needed)

You can still call individual tools directly:

```bash
# Just routing (no execution)
bun ~/.claude/PAI/Tools/EngineRouter.ts route "task"

# Just budget check
bun ~/.claude/PAI/Tools/CrossEngineBudget.ts status

# Just queue status
bun ~/.claude/PAI/Tools/TaskQueue.ts list

# Just dashboard
bun ~/.claude/PAI/Tools/EngineDashboard.ts
```

**But 99% of the time, just use `pai`.**

---

## Cost Savings Examples

### Without `pai` (manual, always PNC):

```bash
claude -p "research X"           # $0.45
claude -p "classify logs"        # $0.38
claude -p "summarize doc"        # $0.22
claude -p "code review"          # $0.95
# Total: $2.00
```

### With `pai` (auto-routed):

```bash
pai "research X"                 # $0.00 (PNG)
pai "classify logs"              # $0.00 (PNO)
pai "summarize doc"              # $0.00 (PNO)
pai "code review"                # $0.95 (PNC)
# Total: $0.95
```

**Savings: $1.05 (52%)**

---

## Tips

### 1. Always `pai`, Never Direct Engine

❌ **Before:**
```bash
claude -p "research topic"
agy -p "research topic"
ollama run gemma2:9b "classify..."
```

✅ **Now:**
```bash
pai "research topic"
pai "classify log"
```

Let the router choose.

---

### 2. Morning Status = Budget Awareness

```bash
pai --status
```

**If >80%:** Be mindful, mostly free engines  
**If >90%:** Critical, free only  
**If <50%:** Use best engines freely

---

### 3. Queue Everything Non-Urgent

```bash
# Instead of blocking
pai "analyze 500 files"  # Blocks for 10 minutes

# Queue it
pai --queue "analyze 500 files"  # Returns immediately
```

---

### 4. Parallel = High Stakes

Only use for:
- Production security audits
- Critical architecture decisions
- Pre-deployment verification

Not for routine work (expensive).

---

### 5. Dashboard = Peace of Mind

Running expensive parallel work?

**Terminal 1:**
```bash
pai --parallel "comprehensive security assessment"
```

**Terminal 2:**
```bash
pai --dashboard
```

Watch cost accumulate, cancel if needed.

---

## Troubleshooting

### "Command not found: pai"

**Solution:**
```bash
# Check symlink
ls -la ~/.local/bin/pai

# Recreate if needed
ln -sf ~/.claude/PAI/Tools/pai ~/.local/bin/pai

# Verify PATH
echo $PATH | grep .local/bin
```

---

### Router selects wrong engine

**Solution:**
```bash
# Force specific engine
pai --engine PNC "task"

# Or tune keywords
# Edit: ~/.claude/PAI/Tools/engine-capability-matrix.json
```

---

### Task fails on selected engine

**The system auto-fails-over to fallback engine.**

If still fails, try:
```bash
# Force different engine
pai --engine PNX "task"
```

---

## Summary

### Before (manual routing):
1. Check what task type
2. Decide which engine
3. Check budget
4. Run engine command
5. Hope you chose right

### Now (automatic):
```bash
pai "task"
```

**That's it.** The system handles everything.

---

## Quick Reference

```bash
# Normal use (99% of time)
pai "your task"

# Morning check
pai --status

# Overnight work
pai --queue "batch task"

# Critical work
pai --parallel "security audit"

# Force engine
pai --engine PNC "task"

# Monitor
pai --dashboard
```

---

**Bottom Line:** Stop calling individual tools. Just use `pai` and let the orchestration system handle routing, budgets, health checks, and execution automatically.

**Start now:**
```bash
pai "research latest AI developments"
```

The system will route it to PNG (free), execute it, and show you the results. No manual work required.
