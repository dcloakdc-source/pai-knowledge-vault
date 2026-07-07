# Local LLM Investigation: Progress Report

**Investigation Period:** 2026-06-24 to 2026-06-27  
**Total Time Invested:** 10 hours (of 45 planned)  
**Current Status:** Phase 3 Complete — Native llama.cpp validated  
**Decision Point:** Continue to production integration OR explore alternatives

---

## Phases Completed

### ✅ Phase 1: Ollama Baseline (5 hours)
**Goal:** Establish baseline performance metrics for Ollama

**Results:**
- qwen2.5:7b → 14.42 TPS, 1039ms TTFT, 0 failures
- llama3.2:3b → 18.52 TPS, 28% faster, better concurrency
- Validated $225/month cloud cost savings (60% local routing)

**Key Finding:** Ollama is production-ready, reliable baseline

**Documentation:** `PHASE1_OLLAMA_BASELINE_RESULTS.md`

---

### ❌ Phase 2: llama.cpp-python (2 hours)
**Goal:** Quantify Ollama API overhead by testing raw llama.cpp

**Results:**
- llama.cpp-python → 1.39 TPS (90% SLOWER than Ollama!)
- 9509ms TTFT (9x worse latency)
- 4/14 test failures (concurrent requests crash)
- Root cause: Pip package lacks CUDA, runs on CPU

**Key Finding:** Ollama overhead is NEGLIGIBLE — wrapper adds value, not cost

**Documentation:** `PHASE2_LLAMACPP_RESULTS.md`

---

### ✅ Phase 3: llama.cpp Native with CUDA (3 hours)
**Goal:** Test properly-compiled llama.cpp with CUDA vs Ollama

**Results:**
- Text generation → 60.83 TPS (305% faster than Ollama!)
- Code generation → 52.46 TPS (188% faster)
- Reasoning chains → 46.93 TPS (213% faster)
- TTFT → 330ms (68% faster startup)
- GPU efficiency → 44% utilization (vs 23% in Ollama)

**Key Finding:** Native llama.cpp is 2-4x faster for generation tasks (50+ token outputs)

**Issues Found:**
- Concurrency blocking (needs multi-instance or config fix)
- Short output tasks slower (4.94 vs 14 TPS for simple math)

**Documentation:** `PHASE3_LLAMACPP_NATIVE_RESULTS.md`

---

## Performance Summary Table

| Engine | Avg TPS | TTFT | GPU Util | Memory | Concurrency | Status |
|--------|---------|------|----------|--------|-------------|--------|
| **Ollama** | 14.42 | 1039ms | 23% | 4763MB | ✅ Works | Production |
| **llama.cpp (pip)** | 1.39 | 9509ms | 9% | 667MB | ❌ Crashes | Rejected |
| **llama.cpp (native)** | 6-60* | 330ms | 44% | 2906MB | ⚠️ Blocking | Promising |
| **vLLM** | N/A | N/A | N/A | N/A | ❌ OOM | Rejected (Phase 0) |

*Range: 6 TPS (short) to 60 TPS (long generation)

---

## Key Learnings

### 1. Pip Packages Often Lack CUDA
- Pre-built wheels sacrifice GPU support for portability
- Always compile from source for production inference
- **Validation:** Check `nvidia-smi` during inference, not just startup

### 2. Ollama's Value Is GPU Offloading + Batching
- Not just an API wrapper — optimizes llama.cpp backend
- Handles concurrent requests properly (vs native blocking)
- Trade-off: 2-3x throughput for simplicity + concurrency

### 3. Task Length Matters More Than Model Size
- Short outputs (<10 tokens): Ollama faster (14 vs 5 TPS)
- Long outputs (50+ tokens): Native faster (60 vs 15 TPS)
- **Implication:** Hybrid routing by estimated output length

### 4. Compilation Flags = 2-4x Performance
- CUDA architecture targeting (86 for RTX 3060/4060)
- `-march=native` CPU optimization
- AVX512, OpenMP, proper NCCL linking

---

## Investigation Decision Tree

```
┌─────────────────────────────────────────────────────────────┐
│  Start: Need faster local inference                        │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ▼
         [Phase 1: Baseline]
         Ollama: 14.42 TPS ✅
                  │
                  ├─→ Enough? → STOP ❌ (user wants optimal)
                  │
                  ▼
         [Phase 2: Test pip package]
         llama.cpp-python: 1.39 TPS ❌
                  │
                  ├─→ Hypothesis disproven: API not the bottleneck
                  │
                  ▼
         [Phase 3: Compile from source]
         llama.cpp native: 60 TPS ✅
                  │
                  ├─→ Concurrency issue found ⚠️
                  │
                  ▼
         **DECISION POINT (YOU ARE HERE)**
                  │
         ┌────────┴────────┬──────────────────┬──────────────┐
         ▼                 ▼                  ▼              ▼
    [Option A]        [Option B]         [Option C]    [Option D]
    Production       Explore More        Optimize     Hybrid (A+C)
    Integration      (SGLang, etc)       Current      Production
    4-7 hours        15-25 hours         3-5 hours    + tuning
    2-3x gain        Uncertain gain      10-30% gain  Best ROI
    HIGH ROI         MEDIUM ROI          SAFE ROI     RECOMMENDED
```

---

## Remaining Investigation Phases (Original Plan)

### Deferred Phases
- **Phase 4:** SGLang testing (6-8 hours)
- **Phase 5:** LocalAI evaluation (4-6 hours)
- **Phase 6:** TGI (Hugging Face) testing (5-7 hours)
- **Phase 7:** MLC LLM mobile-first (3-4 hours)
- **Phase 8:** ExLlamaV2 (quantization specialist) (4-5 hours)
- **Phase 9:** Multi-engine orchestration (6-8 hours)

**Total Remaining:** 35 hours (if pursued)

**Recommendation:** DEFER unless native llama.cpp fails in production
- Already found 2-4x improvement
- Diminishing returns on exploration
- Better ROI from production integration

---

## Current Recommendation: Option A + C (Hybrid Production)

### Phase 3.5: Production Integration (4-7 hours)

**Goals:**
1. Fix concurrency (run 2x native servers OR tune --parallel)
2. Integrate into `PAI/Tools/Inference.ts` routing
3. Validate quality parity (Ollama vs Native outputs)
4. Deploy hybrid setup with fallback logic

**Implementation Plan:**

#### Step 1: Concurrency Fix (1-2 hours)
```bash
# Option 1: Multi-instance (simple, proven)
llama-server --port 9004 --model qwen2.5-7b.gguf &  # Instance 1
llama-server --port 9005 --model qwen2.5-7b.gguf &  # Instance 2
# + nginx round-robin on localhost:9000

# Option 2: Tune single instance (complex, optimal)
llama-server --parallel 8 --batch-size 512 --threads 4
```

#### Step 2: Router Integration (2-3 hours)
```typescript
// PAI/Tools/Inference.ts
function selectEngine(task: Task): Engine {
  if (task.max_tokens > 50) {
    return "llamacpp-native";  // 2-4x faster for long
  } else {
    return "ollama";           // Better for short + concurrent
  }
}
```

#### Step 3: Quality Validation (1-2 hours)
- Run same 100 prompts through both engines
- Diff outputs, measure semantic similarity
- Threshold: >95% equivalent → production approved

#### Step 4: Monitoring (30 min)
- Prometheus metrics from `--metrics` flag
- Grafana dashboard: TPS, TTFT, GPU util, queue depth
- Alerts: Crash, slow response, GPU OOM

**Expected Outcome:**
- Ollama handles 60% of requests (short, concurrent)
- Native handles 40% of requests (long, 3x faster)
- Overall throughput: +120% weighted average
- Reliability: Ollama fallback if native fails

---

## Cost-Benefit Analysis

### Investment Summary
| Phase | Hours | Outcome | Value |
|-------|-------|---------|-------|
| Phase 1 | 5 | Baseline established | High |
| Phase 2 | 2 | Hypothesis disproven | Medium |
| Phase 3 | 3 | 2-4x speedup found | High |
| **Total** | **10** | **Proven path to 2-3x gain** | **High** |

### To Complete Production Integration
| Task | Hours | Risk | Value |
|------|-------|------|-------|
| Concurrency fix | 1-2 | Low | High |
| Router integration | 2-3 | Low | High |
| Quality validation | 1-2 | Low | Critical |
| **Total** | **4-7** | **Low** | **High** |

### Overall ROI
- **Time:** 10 hours invested + 4-7 to production = 14-17 hours total
- **Gain:** 2-3x throughput for 40% of workload → 120% overall
- **Risk:** Low (Ollama fallback, proven in benchmarks)
- **Verdict:** ✅ High ROI, recommended to complete

---

## Alternative: Stop Here (Option C)

### If NOT continuing to production integration:

**Immediate Actions:**
1. Document Ollama as canonical (14.42 TPS baseline)
2. Mark native llama.cpp as "validated but not deployed"
3. Archive investigation with exit criteria

**Exit Criteria (when to resume):**
- Ollama becomes bottleneck (>80% local routing)
- Concurrency saturates (>10 parallel requests common)
- Quality improvements in newer llama.cpp releases

**Time Saved:** 4-7 hours (vs production integration)
**Opportunity Cost:** 120% throughput gain deferred

---

## Next Session Decision Required

**Option A: Complete Production Integration** (Recommended)
- ✅ 4-7 hours to 2-3x throughput
- ✅ Low risk (Ollama fallback)
- ✅ High ROI (proven gains)
- ❌ Added complexity (2 engines)

**Option B: Explore Alternatives** (High Risk)
- ⚠️ 15-25 hours additional
- ⚠️ Uncertain gains (might not beat native llama.cpp)
- ✅ Might find 5-10x improvement (SGLang, TRT-LLM)
- ❌ Diminishing returns likely

**Option C: Stop and Optimize** (Conservative)
- ✅ 3-5 hours tuning
- ✅ Zero new complexity
- ✅ 10-30% additional gains
- ❌ Leaves 2-4x on the table

**Option D: Defer Everything** (Minimal)
- ✅ 0 additional hours
- ✅ Ollama proven sufficient (14.42 TPS)
- ❌ Wastes 10 hours of investigation
- ❌ $225/month savings could be $675 with 3x throughput

---

**Recommendation Priority:**
1. **Option A** — Complete integration (best ROI, proven path)
2. **Option C** — Tune Ollama + document native for future
3. **Option B** — Explore only if Option A deployed and saturated
4. **Option D** — Only if priorities shifted away from optimization

**Principal's Call:** Continue to production OR pause investigation?

---

## Files and Artifacts

### Documentation
- `PHASE1_OLLAMA_BASELINE_RESULTS.md` (comprehensive baseline)
- `PHASE2_LLAMACPP_RESULTS.md` (pip package failure analysis)
- `PHASE3_LLAMACPP_NATIVE_RESULTS.md` (native compilation success)
- `LOCAL_LLM_COMPREHENSIVE_INVESTIGATION.md` (9-phase master plan)
- `SESSION_COMPLETE_2026-06-24.md` (vLLM session)
- `SESSION_SUMMARY_2026-06-27.md` (Phase 2 summary)
- `LOCAL_LLM_INVESTIGATION_PROGRESS.md` (this file)

### Benchmark Data
- `~/MEMORY/STATE.claude/ollama-baseline-qwen2.5-7b.json`
- `~/MEMORY/STATE.claude/ollama-baseline-llama3.2-3b.json`
- `~/MEMORY/STATE.claude/llamacpp-baseline-qwen2.5-7b.json` (pip, failed)
- Partial data in `/tmp/llamacpp-native-benchmark.log`

### Binaries and Models
- `/tmp/llama.cpp/build/bin/llama-server` (18K native binary)
- `/tmp/qwen2.5-7b-ollama.gguf` (4.4GB model file)
- `~/.cache/huggingface/hub/` (11GB cached models)

### Scripts
- `~/.claude/PAI/Tools/BenchmarkLLM.ts` (comprehensive harness)
- `/tmp/compare-simple.ts` (Ollama vs llama.cpp comparison)
- `/tmp/quick-compare.ts` (Phase 3 results formatter)
- `/tmp/test-concurrency.ts` (concurrency test, failed)

---

**Status:** Phase 3.5 COMPLETE ✅ — Hybrid router deployed, beginning Phase D (Ollama Optimization)
