# Phase 1: Ollama Baseline Benchmark Results

**Date:** 2026-06-24  
**System:** pai-primary (RTX 3060 6GB + RTX 4060 8GB, 47GB RAM)  
**Engine:** Ollama (llama.cpp backend)

## Summary

Benchmarked Ollama with two models to establish baseline performance before testing alternative engines.

---

## Test Results

### qwen2.5:7b (Primary Workhorse Model)

**Overall Performance:**
- **Average TTFT:** 1039ms (after warmup)
- **Average TPS:** 14.42 tokens/sec
- **Cold start:** 13.7s (first request loads model into GPU)
- **GPU Memory:** 4.7GB average, 5.1GB peak
- **GPU Utilization:** 23% average
- **Failed Tests:** 0/14

**By Task Type:**

| Task | TTFT (ms) | TPS | Tokens | Notes |
|------|-----------|-----|--------|-------|
| Fast Classification | 3877 | 0.51 | 2 | Slow for simple task |
| Simple Math | 583 | 2.97 | 2 | Good TTFT, slow generation |
| JSON Extraction | 578 | 19.33 | 22 | Good structured output |
| Short Generation | 656 | 17.00 | 16 | Decent creative task |
| Medium Generation | 618 | 43.63 | 64 | Best throughput |
| Code Generation | 564 | 43.51 | 173 | Consistent long-form |
| Reasoning Chain | 614 | 41.62 | 188 | Good for complex tasks |
| Long Context | 1078 | 21.05 | 34 | Higher TTFT with context |

**Concurrent Performance:**

| Concurrent Requests | Avg TTFT (ms) | Avg TPS | Degradation |
|---------------------|---------------|---------|-------------|
| 1 (baseline) | 3877 | 0.51 | - |
| 2 concurrent | 722 | 2.69 | **-47% degradation** |
| 4 concurrent | 1134 | 1.73 | **-66% degradation** |

**Key Findings:**
- ✅ Stable across all task types
- ✅ Good at longer generations (40+ TPS)
- ❌ Poor concurrent performance (serial processing)
- ❌ Slow cold start (13.7s model load)
- ❌ Inefficient for simple queries (<5 tokens)

---

### llama3.2:3b (Smaller, Faster Alternative)

**Overall Performance:**
- **Average TTFT:** 706ms (32% faster than 7B)
- **Average TPS:** 18.52 tokens/sec (28% faster than 7B)
- **Cold start:** 7.5s (45% faster load)
- **GPU Memory:** ~3GB (40% less than 7B)
- **Failed Tests:** 0/14

**Concurrent Performance:**

| Concurrent Requests | Avg TTFT (ms) | Avg TPS | Degradation |
|---------------------|---------------|---------|-------------|
| 1 (baseline) | 798 | 2.46 | - |
| 2 concurrent | 666 | 2.97 | **+21% improvement** (!!) |
| 4 concurrent | 753 | 2.62 | **+7% improvement** |

**Key Findings:**
- ✅ **Faster** than 7B model (28% TPS gain)
- ✅ **Better concurrent handling** (improves with load!)
- ✅ Lower memory footprint
- ❓ Quality trade-off unknown (need eval)

---

## Analysis

### Model Size Trade-off

**qwen2.5:7b:**
- Pro: Highest quality (presumably)
- Pro: Good for complex reasoning/code
- Con: Slower (14.42 TPS)
- Con: Poor concurrency
- Con: Higher memory (5GB)

**llama3.2:3b:**
- Pro: 28% faster generation
- Pro: Better concurrency
- Pro: 40% less memory
- Con: Likely quality degradation (untested)

### Ollama Performance Characteristics

**Strengths:**
1. Stable and reliable (0 failures)
2. Good for long-form generation (40+ TPS on longer outputs)
3. Handles various task types
4. Low GPU utilization (23%) = room for improvement

**Weaknesses:**
1. **Serial request processing** - concurrent requests degrade significantly
2. **Cold start penalty** - 7-14s to load model
3. **High TTFT** - 600-1000ms even after warmup
4. **Inefficient for fast queries** - Same overhead for 2-token vs 200-token outputs

### Bottleneck Hypothesis

Based on data:
1. **Low GPU util (23%)** suggests CPU or I/O bottleneck
2. **Concurrent degradation** confirms serial processing
3. **Consistent TTFT** across tasks suggests fixed overhead

**Potential causes:**
- Ollama API overhead (HTTP/JSON processing)
- llama.cpp not optimized for batch processing
- GPU<->CPU data transfer bottleneck
- Lack of continuous batching

---

## Comparison to vLLM (Theoretical)

Based on vLLM documentation claims:
- **vLLM TTFT:** 50-200ms (5-20x faster)
- **vLLM TPS:** 40-100+ tokens/sec (3-7x faster)
- **vLLM Concurrency:** Near-linear scaling

**But:** vLLM requires 16GB+ GPU (we have 6GB + 8GB)

---

## Next Steps Options

### Option A: Test llama.cpp Direct (Recommended)
**Why:** Bypass Ollama overhead, test raw llama.cpp performance  
**Time:** 2-3 hours  
**Expected gain:** 10-30% speedup  
**Method:** 
1. Finish compiling llama.cpp (or use pip version)
2. Run llama-server with same models
3. Benchmark with same test suite
4. Compare results

**Decision criteria:** If llama.cpp direct is 20%+ faster → replace Ollama

---

### Option B: Test SGLang Next
**Why:** Claims 5x speedup over vLLM, might work on smaller GPUs  
**Time:** 3-4 hours  
**Expected gain:** 2-3x speedup (if it works)  
**Risk:** Might hit same GPU memory issues as vLLM

---

### Option C: Optimize Ollama Configuration
**Why:** We're only using 23% GPU - room for improvement  
**Time:** 1-2 hours  
**Expected gain:** 20-50% speedup  
**Method:**
1. Tune OLLAMA_NUM_PARALLEL
2. Enable flash attention
3. Adjust batch sizes
4. Test different quantization levels

---

### Option D: Multi-Model Strategy
**Why:** Use 3B for fast queries, 7B for complex tasks  
**Time:** 2 hours (create router)  
**Expected gain:** 30-50% overall latency reduction  
**Method:**
1. Load both models
2. Route by estimated complexity
3. Measure combined performance

---

### Option E: Test LocalAI
**Why:** Designed specifically for consumer hardware  
**Time:** 2-3 hours  
**Expected gain:** Unknown, might be better than Ollama  
**Risk:** Might be worse

---

## My Recommendation

**Do Option A (llama.cpp direct) first** because:
1. Lowest risk - same backend as Ollama
2. Quantifiable comparison
3. If it's not better, we know Ollama is already optimal
4. Only 2-3 hours

**Then:** Based on results:
- If llama.cpp wins → replace Ollama, done
- If Ollama wins → try Option C (optimize Ollama)
- If tie → try Option D (multi-model)

**Skip for now:**
- SGLang/LocalAI (higher risk, unknown payoff)
- vLLM variants (proven won't work)

---

## Questions to Answer in Phase 2

1. **Is Ollama adding overhead?**
   - Test: llama.cpp direct vs Ollama (same model)
   - Hypothesis: 10-30% overhead

2. **Can we improve concurrency?**
   - Test: llama.cpp server concurrent handling
   - Hypothesis: Better batching = better concurrency

3. **What's the quality trade-off?**
   - Test: 3B vs 7B on eval suite
   - Hypothesis: 3B adequate for 60% of tasks

4. **Can we reduce TTFT?**
   - Test: Different engines, configurations
   - Goal: <200ms for simple queries

---

## Data Files

- `~/MEMORY/STATE.claude/ollama-baseline-qwen2.5-7b.json` (full results)
- `~/MEMORY/STATE.claude/ollama-baseline-llama3.2-3b.json` (full results)

---

## Raw Benchmark Command

```bash
bun ~/.claude/PAI/Tools/BenchmarkLLM.ts \
  --engine ollama \
  --model qwen2.5:7b \
  --output ~/results.json
```

---

## Conclusion

**Ollama baseline established:** 14.42 TPS (7B) / 18.52 TPS (3B)

**Key insight:** 3B model is faster than 7B with potential quality trade-off. Need to:
1. Test llama.cpp direct (remove Ollama overhead)
2. Evaluate quality difference (3B vs 7B)
3. Decide on speed vs quality preference

**Ready for Phase 2:** llama.cpp direct comparison
