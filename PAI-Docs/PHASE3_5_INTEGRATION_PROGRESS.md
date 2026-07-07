# Phase 3.5: Production Integration Progress

**Date:** 2026-06-27  
**Duration:** 2 hours (of 4-7 planned)  
**Status:** In Progress — Core components built, debugging needed

---

## Goal

Integrate native llama.cpp into production alongside Ollama with intelligent routing:
- Short outputs (<50 tokens) → Ollama (proven concurrent handling)
- Long outputs (≥50 tokens) → Native llama.cpp (2-4x faster)

---

## Components Built

### 1. Load Balancer ✅
**File:** `PAI/Tools/LlamaCppLoadBalancer.ts`

**Features:**
- Round-robin distribution across multiple llama-server instances
- Health checking every 10s with 3-failure threshold
- Automatic failover to healthy instances
- Backend tracking via `X-LB-Backend` header
- Unified endpoint at :9010 (or configurable port)

**Status:** Code complete, tested health checks work

**Issue Found:** llama-server instances hang under concurrent load despite `--parallel 4` flag

### 2. Hybrid Router ✅
**File:** `PAI/Tools/HybridInference.ts`

**Features:**
- Intelligent routing based on estimated output length
- Automatic fallback if primary backend fails
- CLI interface for testing
- Export function for programmatic use
- Routing threshold: 50 tokens (configurable)

**Status:** Code complete, Ollama path tested successfully (14.8s for short classification)

**Issue Found:** llama.cpp `/completion` endpoint hangs on requests

###3. Systemd Service Files ✅
**Files:** `/tmp/llamacpp-native-1.service`, `/tmp/llamacpp-native-2.service`

**Features:**
- Auto-restart on crash
- Resource limits (8GB RAM, 400% CPU)
- Separate instances on ports 9004, 9005
- Journal logging

**Status:** Templates created, not yet installed to `/etc/systemd/system/`

---

## Test Results

### Ollama Path (Working)
```bash
bun HybridInference.ts --prompt "Classify..." --max-tokens 5
# ✅ Routed to Ollama
# ✅ 14.8s latency
# ✅ Correct output
```

### Native llama.cpp Path (Blocked)
```bash
bun HybridInference.ts --prompt "Write essay..." --max-tokens 150
# ❌ Timeout after 60s
# ❌ Server process shows 99.7% CPU (stuck)
# ❌ Health endpoint responds, but /completion hangs
```

---

## Root Cause Analysis

### Issue: llama-server Hanging Under Load

**Symptoms:**
1. Health endpoint (`/health`) responds instantly
2. Completion endpoint (`/completion`) hangs indefinitely
3. CPU pegged at 100% on single core
4. Process does not crash, just stops responding to new requests

**Hypothesis:**
1. **Single-threaded queue** — despite `--parallel 4`, server processes requests sequentially
2. **Previous request stuck** — if one request doesn't complete, queue blocks
3. **Context size issue** — might be trying to load full 8192 context unnecessarily
4. **Streaming vs non-streaming** — we're using non-streaming mode, might need streaming

**Evidence:**
- Server was running for 42+ hours (started Jun 28, now Jun 27... wait, time anomaly?)
- Process uptime suggests it survived previous benchmark timeout
- Multiple incomplete requests likely queued

**Next Steps to Debug:**
1. Restart llama-server fresh (kill and relaunch)
2. Test single simple request with curl
3. Try streaming mode instead of non-streaming
4. Reduce `--ctx-size` from 8192 to 2048
5. Test with `--parallel 1` to isolate threading issues

---

## Multi-Instance Setup

### GPU Memory Availability ✅
- **GPU 0 (RTX 3060):** 2.9GB free, instance 1 using ~3GB
- **GPU 1 (RTX 4060):** 4.8GB free, can fit instance 2

### Attempted Configuration
```bash
# Instance 1 on GPU 0, port 9004
/tmp/llama.cpp/build/bin/llama-server --model ... --port 9004

# Instance 2 on GPU 1, port 9005
CUDA_VISIBLE_DEVICES=1 /tmp/llama.cpp/build/bin/llama-server --model ... --port 9005
```

**Status:** Instance 2 launched successfully, both showed healthy, but both hang on completion requests

---

## Integration Points

### Where Hybrid Router Will Plug In

**Option A: Replace calls in existing tools**
```typescript
// Before:
const result = await inferenceOllama({...});

// After:
import { hybridInfer } from "./HybridInference";
const result = await hybridInfer({...});
```

**Option B: Add to Inference.ts routing matrix**
```json
{
  "code_generation": {
    "primary": "hybrid/qwen2.5:7b",
    "fallback": "ollama/qwen2.5:7b"
  }
}
```

**Recommendation:** Option A initially (surgical, testable), then Option B once proven

---

## Remaining Work

### Immediate (Must Fix)
- [ ] Debug llama-server hanging issue
  - [ ] Restart with clean state
  - [ ] Test with simple curl request
  - [ ] Try streaming mode
  - [ ] Reduce context size
  - [ ] Monitor with nvidia-smi during request

### Short-term (This Phase)
- [ ] Get single llama-server instance working reliably
- [ ] Test HybridInference.ts end-to-end (both paths)
- [ ] Quality validation: Compare outputs Ollama vs Native
- [ ] Load test: 10 sequential requests, no hangs
- [ ] Document configuration tuning (threads, context, batch size)

### Medium-term (Before Production)
- [ ] Multi-instance if concurrency needed (currently blocked on single-instance stability)
- [ ] Systemd service installation and auto-start
- [ ] Monitoring dashboard (Prometheus + Grafana)
- [ ] Integration into 2-3 high-volume PAI tools
- [ ] 1-week burn-in period with fallback to Ollama

---

## Decision Point

**Continue debugging (2-3 hours)?**
- Pro: We're close — server compiles, runs, health check works
- Pro: 2-4x speedup proven in Phase 3 benchmarks
- Con: Unknown time to debug hanging issue
- Con: Might hit more issues (concurrency, stability)

**OR Defer and document?**
- Pro: Ollama works perfectly now (14.42 TPS, 0 failures)
- Pro: 10 hours investigation complete, gains documented
- Pro: Can return when time allows
- Con: Leaves 2-4x performance on table
- Con: Wastes 2 hours of Phase 3.5 work

**Hybrid approach (RECOMMENDED):**
1. **Immediate:** Restart llama-server clean, test ONE simple request
2. **IF works:** Complete HybridInference testing (30 min)
3. **IF still hangs:** Document issue, defer to Phase 4 (separate debugging session)
4. **Move forward:** Either with working hybrid OR with Ollama-only (proven stable)

---

## Time Accounting

| Phase | Planned | Actual | Status |
|-------|---------|--------|--------|
| Phase 1: Ollama baseline | N/A | 5 hours | Complete ✅ |
| Phase 2: llama.cpp-python | N/A | 2 hours | Complete ❌ (failed) |
| Phase 3: Native compilation | N/A | 3 hours | Complete ✅ (2-4x faster) |
| **Phase 3.5: Integration** | **4-7 hours** | **2 hours** | **In Progress** |
| → Load balancer | 1 hour | 0.5 hours | Code complete ⚠️ |
| → Hybrid router | 1 hour | 0.5 hours | Code complete ⚠️ |
| → Debugging | 1 hour | 1 hour | Blocked on hanging |
| → Testing/validation | 1-2 hours | 0 hours | Not started |
| → Production deployment | 0.5-1 hour | 0 hours | Not started |

**Total invested:** 12 hours (10 previous + 2 this session)  
**Total remaining (estimated):** 2-5 hours to complete OR 0 hours to defer

---

## Files Created

### Production Code
- `PAI/Tools/LlamaCppLoadBalancer.ts` (164 lines, complete)
- `PAI/Tools/HybridInference.ts` (239 lines, complete, partial testing)

### Configuration
- `/tmp/llamacpp-native-1.service` (systemd template)
- `/tmp/llamacpp-native-2.service` (systemd template)

### Documentation
- This file

---

## Next Session Actions

**Path A: Debug and Complete** (2-3 hours)
1. Kill stuck llama-server: `pkill -9 llama-server`
2. Restart fresh with reduced config:
   ```bash
   /tmp/llama.cpp/build/bin/llama-server \
     --model /tmp/qwen2.5-7b-ollama.gguf \
     --host 127.0.0.1 \
     --port 9004 \
     --n-gpu-layers 99 \
     --ctx-size 2048 \  # Reduced from 8192
     --parallel 1 \      # Disable parallel until stable
     --metrics
   ```
3. Test with curl: `curl -X POST http://localhost:9004/completion -d '{"prompt":"Hello","n_predict":10}'`
4. If works → test HybridInference.ts
5. If works → quality validation (10 prompts, compare outputs)
6. If works → deploy to production

**Path B: Defer and Document** (30 min)
1. Update `LOCAL_LLM_INVESTIGATION_PROGRESS.md` with Phase 3.5 status
2. Mark as "Deferred pending llama-server stability investigation"
3. Document known issue + reproduction steps
4. Archive for future session
5. Return to using Ollama (proven stable)

**Path C: Simplified Alternative** (1-2 hours)
1. Skip native llama.cpp for now
2. Optimize Ollama configuration instead:
   - Test Q5_K_M quantization (higher quality)
   - Tune `num_thread`, `num_gpu` parameters
   - Test flash-attention flags
3. Expected gain: 10-30% improvement
4. Zero new complexity, build on proven foundation

---

**Recommendation:** Path A for one more debugging session (2-3 hours cap), then Path B if still blocked.

**Rationale:**
- We've proven 2-4x gains exist (Phase 3 benchmarks)
- Server compiles and runs (not a dead end)
- Hanging might be simple config issue (--parallel, --ctx-size)
- 2-3 hour time-box is reasonable given potential 2-4x payoff
- If blocked after time-box, defer cleanly with full documentation

---

**Status:** Awaiting principal decision — continue debugging OR defer Phase 3.5?
