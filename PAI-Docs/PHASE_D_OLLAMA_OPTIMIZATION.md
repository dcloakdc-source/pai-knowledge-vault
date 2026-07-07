# Phase D: Ollama Optimization

**Date:** 2026-06-30  
**Duration:** 3-5 hours (estimated)  
**Goal:** Extract 10-30% more performance from Ollama baseline

---

## Current Baseline

From Phase 1 (2026-06-24):
- **Model:** qwen2.5:7b (Q4_K_M quantization)
- **TPS:** 14.42 average
- **TTFT:** 1039ms average
- **GPU Usage:** 23% utilization, 4763MB memory
- **Failures:** 0/14 tests
- **Concurrency:** Degrades under load (-66% at 4x)

---

## Optimization Vectors

### 1. Quantization Testing
**Hypothesis:** Higher quantization (Q5, Q8) may improve quality/speed at memory cost

**Models to test:**
- Q4_K_M (current baseline)
- Q5_K_M (balanced quality/size)
- Q8_0 (near-FP16 quality, 2x size)

**Metrics:**
- TPS (tokens per second)
- Quality (same prompts, compare outputs)
- GPU memory usage
- Model size on disk

### 2. Ollama Configuration
**Hypothesis:** Threading and GPU layer tuning can improve throughput

**Parameters to test:**
- `num_thread` (4, 8, 16, auto)
- `num_gpu` (99, 40, 20 - partial offload)
- `num_ctx` (2048, 4096, 8192 - context window)
- `num_batch` (512, 1024, 2048 - batch size)

### 3. Model Size Trade-off
**Hypothesis:** Smaller model (3B) might be faster with acceptable quality loss

**Comparison:**
- llama3.2:3b (18.52 TPS from Phase 1)
- qwen2.5:7b (14.42 TPS baseline)
- Quality delta measurement

### 4. Flash Attention
**Hypothesis:** Flash attention flag improves memory efficiency

**Test:**
- Standard attention (default)
- Flash attention v2 (if supported)

---

## Test Plan

### Test 1: Quantization Comparison (1-2 hours)

**Setup:**
1. Download Q5_K_M and Q8_0 versions of qwen2.5:7b
2. Run same benchmark suite from Phase 1
3. Compare TPS, quality, memory

**Commands:**
```bash
# Download quantizations
ollama pull qwen2.5:7b-q5-k-m
ollama pull qwen2.5:7b-q8-0

# Benchmark each
bun BenchmarkLLM.ts --model qwen2.5:7b-q4-k-m --output q4.json
bun BenchmarkLLM.ts --model qwen2.5:7b-q5-k-m --output q5.json
bun BenchmarkLLM.ts --model qwen2.5:7b-q8-0 --output q8.json

# Compare results
bun CompareQuantizations.ts q4.json q5.json q8.json
```

**Expected outcome:**
- Q5_K_M: +5-10% quality, -5% speed, +30% memory
- Q8_0: +10-15% quality, -10-15% speed, +100% memory

**Decision criteria:**
- Use Q5_K_M if quality gain > 10% AND speed loss < 10%
- Stick with Q4_K_M otherwise

### Test 2: Configuration Tuning (1-2 hours)

**Setup:**
1. Test different `num_thread` values
2. Test partial GPU offloading
3. Test context window sizes
4. Measure TPS for each configuration

**Test matrix:**
```
num_thread: [4, 8, 16, auto]
num_gpu: [99, 40, 20]
num_ctx: [2048, 4096, 8192]
num_batch: [512, 1024, 2048]
```

**Approach:** One variable at a time from baseline

**Commands:**
```bash
# Test threading
bun TestOllamaConfig.ts --num-thread 4
bun TestOllamaConfig.ts --num-thread 8
bun TestOllamaConfig.ts --num-thread 16

# Test GPU layers
bun TestOllamaConfig.ts --num-gpu 99  # baseline
bun TestOllamaConfig.ts --num-gpu 40
bun TestOllamaConfig.ts --num-gpu 20

# etc...
```

**Expected outcome:**
- Optimal threading: 8-16 cores
- Full GPU offload likely best (99 layers)
- Larger context = slower but more capable

### Test 3: 3B vs 7B Quality Delta (1 hour)

**Setup:**
1. Select 20 diverse prompts (classification, math, generation, code)
2. Run through llama3.2:3b and qwen2.5:7b
3. Human review of quality differences
4. Build task routing matrix

**Prompts:**
- 5x classification (short, factual)
- 5x math/reasoning
- 5x generation (creative, explanatory)
- 5x code generation

**Quality scoring:**
- 0 = Wrong/useless
- 1 = Acceptable
- 2 = Good
- 3 = Excellent

**Decision criteria:**
- If 3B scores ≥80% of 7B across tasks → route short tasks to 3B
- If 3B comparable on classification → route all <20 token tasks to 3B

### Test 4: Concurrency Optimization (30 min)

**Setup:**
1. Re-test concurrent requests (2x, 4x, 8x)
2. Try with optimized config from Test 2
3. Measure degradation

**Baseline (Phase 1):**
- 1x: 14.42 TPS
- 2x: 17.45 TPS (+21%)
- 4x: 4.85 TPS (-66%)

**Goal:** Reduce 4x degradation to <50%

---

## Deliverables

1. **Benchmark Results**
   - Quantization comparison table
   - Configuration tuning results
   - 3B vs 7B quality matrix
   - Concurrency performance

2. **Optimal Configuration**
   - Best quantization level
   - Recommended Ollama parameters
   - Task → model routing rules

3. **Documentation**
   - Tuning guide for future models
   - Performance expectations per config
   - Integration recommendations

4. **Code**
   - TestOllamaConfig.ts (configuration tester)
   - CompareQuantizations.ts (result comparison)
   - Updated inference-matrix.json

---

## Success Metrics

- [ ] 10-30% TPS improvement via tuning
- [ ] Quality parity or better maintained
- [ ] Concurrency degradation reduced
- [ ] Clear routing rules (3B vs 7B)
- [ ] Documented optimal configuration

---

## Status: READY TO BEGIN

Proceeding with Test 1: Quantization Comparison
