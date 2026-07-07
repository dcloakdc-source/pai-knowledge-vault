# Soft Launch Plan: Hybrid Inference Integration

**Date:** 2026-06-30  
**Duration:** 1 week monitoring  
**Goal:** Validate hybrid router in production with real workload

---

## Target Integration Points

### Criteria for Selection
- High volume (frequent inference calls)
- Mix of short and long outputs (tests both paths)
- Non-critical (can rollback easily)
- Easy to instrument (add logging)

### Selected Tools (3)

#### 1. PAI/Tools/Inference.ts (Primary Target)
**Why:** Central inference router used by many PAI tools

**Current state:**
- Routes to Ollama, Gemini, Claude, OpenAI
- Has fallback logic
- Used by skills and agents

**Integration:**
```typescript
// Add hybrid routing for local models
import { hybridInfer } from "./HybridInference";

async function inferenceOllama(options: InferenceOptions, modelName: string) {
  // NEW: Use hybrid router instead of direct Ollama
  const result = await hybridInfer({
    prompt: options.userPrompt,
    systemPrompt: options.systemPrompt,
    maxTokens: options.options?.num_predict || 100,
    temperature: 0.7,
    model: modelName
  });
  
  return {
    success: result.success,
    output: result.output,
    latencyMs: result.latencyMs,
    model: `${result.backend}/${modelName}`
  };
}
```

**Monitoring:**
- Log backend used (ollama vs llamacpp-native)
- Track latency per backend
- Count fallback occurrences
- Alert if latency >10s

#### 2. Skills/Research/Tools/*.ts (Secondary Target)
**Why:** Heavy users of inference, diverse prompt lengths

**Current state:**
- Multiple research tools call Ollama directly
- Mix of classification (short) and summarization (long)

**Integration:**
```typescript
// In research tools that generate summaries
import { hybridInfer } from "../../Tools/HybridInference";

// Replace direct Ollama calls with hybrid
const summary = await hybridInfer({
  prompt: `Summarize this content:\n\n${content}`,
  maxTokens: 200,  // Long output → routes to native
  temperature: 0.7
});
```

**Monitoring:**
- Compare summary quality (human review sample)
- Measure latency improvements
- Track which backend used per task type

#### 3. PAI/Tools/training/archetype-serve.ts (Tertiary Target)
**Why:** Local archetype routes, controlled workload

**Current state:**
- Uses Ollama for local archetype inference
- Predictable prompt patterns
- Easy to A/B test

**Integration:**
```typescript
// Add hybrid option to archetype serving
export async function serveArchetype(
  archetype: string,
  input: string,
  useHybrid: boolean = true  // Feature flag
) {
  if (useHybrid) {
    return await hybridInfer({
      prompt: input,
      maxTokens: estimateTokens(archetype),
      temperature: 0.7
    });
  } else {
    // Original Ollama path
  }
}
```

**Monitoring:**
- A/B test: 50% hybrid, 50% direct Ollama
- Compare latency distributions
- Validate quality equivalence

---

## Implementation Steps

### Phase 1: Preparation (30 min)
- [x] Model copied to permanent location
- [x] Systemd service created
- [x] Post-reboot script ready
- [ ] Reboot system
- [ ] Run PostRebootSetup.sh
- [ ] Verify both backends healthy

### Phase 2: Integration (2 hours)
1. **Inference.ts** (1 hour)
   - Import HybridInference
   - Add hybrid routing to inferenceOllama()
   - Add backend logging
   - Test with sample prompts

2. **Research tools** (30 min)
   - Pick 2-3 high-volume research scripts
   - Replace Ollama calls with hybridInfer()
   - Add latency logging

3. **Archetype serve** (30 min)
   - Add hybrid option with feature flag
   - Default to false initially
   - Test with known archetypes

### Phase 3: Testing (1 hour)
```bash
# Test each integration point
cd ~/.claude/PAI/Tools

# Test 1: Inference.ts
bun Inference.ts --task fast_classification "Test prompt"

# Test 2: Research tool
bun skills/Research/Tools/SomeResearchTool.ts

# Test 3: Archetype
bun training/archetype-serve.ts --archetype planning "Test"
```

### Phase 4: Monitoring (1 week)

**Daily checks:**
```bash
# Check backend usage distribution
grep "HybridInfer" ~/.claude/logs/* | grep "Routing to" | \
  awk '{print $NF}' | sort | uniq -c

# Check average latency per backend
grep "latencyMs" ~/.claude/logs/* | \
  awk -F'latencyMs":' '{print $2}' | \
  awk -F',' '{print $1}' | \
  awk '{sum+=$1; count++} END {print "Avg:", sum/count "ms"}'

# Check for fallback occurrences
grep "fallback from" ~/.claude/logs/* | wc -l
```

**Weekly review:**
- Compare latency: hybrid vs baseline
- Review any errors/timeouts
- Check quality (sample 10 outputs)
- Decide: rollout or rollback

---

## Success Criteria

### Must Have ✅
- [ ] Zero crashes or hangs
- [ ] Latency < baseline for long outputs
- [ ] Quality parity (sample review passes)
- [ ] Fallback logic triggers correctly

### Should Have ✅
- [ ] 20-40% of requests route to native
- [ ] 2x+ speedup for those requests
- [ ] No user-visible regressions
- [ ] Monitoring data confirms routing logic

### Nice to Have
- [ ] Reduced cloud API costs
- [ ] Improved user experience (faster)
- [ ] Clear data for full rollout decision

---

## Rollback Plan

### If Issues Detected

**Immediate rollback:**
```bash
# Revert Inference.ts
git checkout HEAD~1 PAI/Tools/Inference.ts

# Disable hybrid in other tools
# Change useHybrid flag to false

# Restart services
sudo systemctl restart ollama
```

**Investigate:**
- Check logs for errors
- Test native llama-server directly
- Verify GPU memory not exhausted
- Check system load

**Re-attempt after fixes**

---

## Post-Soft Launch

### If Successful → Full Rollout

1. **Enable for all tools** (2-3 hours)
   - Update all Ollama callers to use hybrid
   - Remove feature flags
   - Set as default in Inference.ts

2. **Documentation** (1 hour)
   - Update PAI docs with hybrid usage
   - Add troubleshooting guide
   - Document performance gains

3. **Optimization** (ongoing)
   - Fine-tune 50-token threshold
   - Add task-specific routing hints
   - Monitor for new optimization opportunities

### If Issues Found → Iterate

1. **Debug specific failures**
2. **Tune routing threshold**
3. **Fix quality issues**
4. **Re-test in controlled environment**
5. **Attempt soft launch again**

---

## Timeline

**Day 0 (Today):**
- [x] Complete investigation (15 hours)
- [x] Create soft launch plan
- [ ] Reboot system (waiting on principal)
- [ ] Run post-reboot setup

**Day 1 (After reboot):**
- [ ] Integrate into 3 target tools
- [ ] Test all integration points
- [ ] Begin monitoring

**Days 2-7:**
- [ ] Daily monitoring checks
- [ ] Review logs and metrics
- [ ] Sample quality checks
- [ ] Document any issues

**Day 8 (Decision point):**
- [ ] Review 1-week data
- [ ] Decision: Full rollout OR rollback OR iterate

---

## Monitoring Dashboard (Manual)

### Daily Checklist

```bash
# 1. Backend distribution
echo "=== Backend Usage ==="
grep "Routing to" /tmp/hybrid-*.log 2>/dev/null | \
  awk '{print $NF}' | sort | uniq -c || echo "No logs yet"

# 2. Average latency
echo "=== Average Latency ==="
for backend in ollama llamacpp; do
  echo -n "$backend: "
  grep "$backend.*latencyMs" /tmp/hybrid-*.log 2>/dev/null | \
    awk -F'latencyMs":' '{print $2}' | awk -F',' '{print $1}' | \
    awk '{sum+=$1; count++} END {if(count>0) print sum/count "ms"; else print "N/A"}'
done

# 3. Errors
echo "=== Errors ==="
grep -i "error\|failed\|timeout" /tmp/hybrid-*.log 2>/dev/null | wc -l

# 4. System health
echo "=== System Health ==="
uptime
nvidia-smi --query-gpu=index,utilization.gpu,memory.used --format=csv,noheader
```

---

## Status: Ready for Reboot

All preparation complete. Waiting for system reboot to proceed with soft launch.

**Next action:** Reboot system, then run `~/.claude/PAI/Tools/PostRebootSetup.sh`
