# Comprehensive Local LLM Investigation Plan

**Goal:** Find the optimal local inference stack for pai-primary, anticipating local becoming primary source for all AI activities.

**Hardware:** RTX 3060 Laptop 6GB + RTX 4060 8GB, 47GB RAM, CUDA 12.0

**Success Criteria:**
1. Maximum throughput (tokens/sec)
2. Minimum latency (time to first token)
3. Best memory efficiency (largest model that fits)
4. Stability (24/7 operation)
5. API compatibility (OpenAI-compatible preferred)
6. Ease of management (systemd integration, monitoring)

---

## Phase 1: Baseline Measurement (Ollama)

**Current State - Measure Before Changing**

### Tests to Run
1. **Memory efficiency**
   - GPU memory per model size
   - How many models can load simultaneously
   - Memory overhead vs raw model size

2. **Performance benchmarks**
   - Time to first token (TTFT)
   - Tokens per second (generation speed)
   - Concurrent request handling
   - Context length impact on performance

3. **Model quality at different quants**
   - Q4_K_M vs Q6_K vs Q8_0
   - Quality degradation measurement
   - Speed vs quality tradeoff

4. **Configuration tuning**
   - OLLAMA_NUM_PARALLEL impact
   - OLLAMA_MAX_LOADED_MODELS impact  
   - OLLAMA_FLASH_ATTENTION impact
   - Context caching effectiveness

**Deliverable:** `OLLAMA_BASELINE_BENCHMARK.md` with hard numbers

---

## Phase 2: llama.cpp Direct (Ollama's Backend)

**Hypothesis:** Ollama adds overhead, direct llama.cpp might be faster

### Installation
```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
make LLAMA_CUDA=1
```

### Tests
1. **Same models, direct comparison**
   - Load qwen2.5:7b GGUF directly
   - Measure TTFT, tokens/sec
   - Compare to Ollama's performance

2. **Advanced quantization**
   - Try quantization formats Ollama doesn't expose
   - IQ4_XS, IQ3_M, etc.
   - Measure quality vs speed

3. **Server mode**
   ```bash
   ./llama-server --model qwen2.5-7b.gguf --port 8080 --n-gpu-layers 99
   ```
   - OpenAI-compatible API
   - Performance vs Ollama API
   - Concurrent request handling

4. **Memory optimization flags**
   - Flash attention
   - KV cache optimization
   - Batch size tuning

**Deliverable:** `LLAMACPP_DIRECT_BENCHMARK.md`

**Decision Point:** If llama.cpp direct is significantly faster, consider replacing Ollama

---

## Phase 3: Alternative Serving Engines

### 3A. LocalAI (OpenAI-compatible, consumer-focused)

**Why:** Designed specifically for consumer hardware, multiple backend support

**Installation:**
```bash
docker run -d --name localai \
  --gpus all \
  -v ~/.cache/huggingface:/models \
  -p 8080:8080 \
  localai/localai:latest-gpu-nvidia-cuda-12
```

**Tests:**
- Same model (Qwen2.5-7B)
- Performance comparison
- Memory efficiency
- Multi-model support
- API compatibility

**Deliverable:** `LOCALAI_BENCHMARK.md`

---

### 3B. SGLang (Claims 5x speedup over vLLM)

**Why:** Structured Generation Language, optimized for efficiency

**Installation:**
```bash
pip install "sglang[all]"
```

**Tests:**
```bash
python -m sglang.launch_server \
  --model Qwen/Qwen2.5-7B-Instruct \
  --port 30000 \
  --mem-fraction-static 0.8
```

- Benchmark vs vLLM's failed attempts
- Check if memory management better
- RadixAttention effectiveness
- Structured output performance

**Deliverable:** `SGLANG_BENCHMARK.md`

---

### 3C. Aphrodite Engine (vLLM fork, optimized)

**Why:** vLLM fork focused on consumer GPUs, better quantization support

**Installation:**
```bash
pip install aphrodite-engine
```

**Tests:**
- AWQ/GPTQ/EXL2 support
- Memory efficiency improvements
- API compatibility
- Speed comparison

**Deliverable:** `APHRODITE_BENCHMARK.md`

---

### 3D. Text Generation Inference (Hugging Face TGI)

**Why:** Production-grade, used by HF internally

**Installation:**
```bash
docker run -d --name tgi \
  --gpus all \
  -v ~/.cache/huggingface:/data \
  -p 8081:80 \
  ghcr.io/huggingface/text-generation-inference:latest \
  --model-id Qwen/Qwen2.5-7B-Instruct \
  --quantize awq
```

**Tests:**
- Quantization support (AWQ, GPTQ, bitsandbytes)
- Continuous batching effectiveness
- Tensor parallelism across 2 GPUs
- Production stability

**Deliverable:** `TGI_BENCHMARK.md`

---

### 3E. MLC LLM (Apache TVM-based)

**Why:** Compiles models to optimized kernels, claims best memory efficiency

**Installation:**
```bash
pip install mlc-llm mlc-ai-nightly
```

**Tests:**
- Model compilation to optimized format
- Memory usage vs raw PyTorch
- Speed after compilation
- Quantization options (4-bit, 3-bit)

**Deliverable:** `MLC_LLM_BENCHMARK.md`

---

### 3F. ExLlamaV2 (GPTQ-focused)

**Why:** Fastest GPTQ inference engine

**Installation:**
```bash
git clone https://github.com/turboderp/exllamav2
pip install -e exllamav2
```

**Tests:**
- EXL2 quantization (better than AWQ?)
- Speed claims (2x+ faster than transformers)
- Memory efficiency
- Multi-GPU support

**Deliverable:** `EXLLAMAV2_BENCHMARK.md`

---

## Phase 4: Multi-Engine Architecture

**Hypothesis:** Different engines excel at different tasks

### Strategy: Route by Task Type

```
┌─────────────────────────────────────┐
│         PAI Inference Router        │
└─────────────────────────────────────┘
                  │
        ┌─────────┴─────────┐─────────┐
        │                   │         │
   ┌────▼────┐       ┌──────▼──┐  ┌──▼─────┐
   │ llama.cpp│       │ SGLang  │  │  TGI   │
   │ (fast)   │       │(struct) │  │(batch) │
   └──────────┘       └─────────┘  └────────┘
```

**Tests:**
1. **Task classification**
   - Fast single queries → llama.cpp
   - Structured output → SGLang
   - Batch processing → TGI
   
2. **Routing overhead**
   - Decision latency
   - Load balancing
   - Failure handling

3. **Combined throughput**
   - Can we saturate both GPUs?
   - Better utilization than single engine?

**Deliverable:** `MULTI_ENGINE_ARCHITECTURE.md`

---

## Phase 5: Quantization Deep Dive

**Goal:** Find optimal quality/speed/memory balance

### Formats to Test
1. **GGUF variants** (llama.cpp)
   - Q4_K_M, Q4_K_S, Q5_K_M, Q6_K, Q8_0
   - IQ4_XS, IQ3_M (imatrix quants)
   
2. **AWQ** (Activation-aware Weight Quantization)
   - 4-bit, group size 128
   - Tried with vLLM, test with others

3. **GPTQ** (Post-training quantization)
   - 4-bit, 3-bit
   - Group size impact

4. **EXL2** (ExLlamaV2 format)
   - Variable bitrate
   - Quality claims

5. **bitsandbytes** (Runtime quantization)
   - INT8, FP4, NF4
   - Dynamic vs static

### Quality Evaluation
- MMLU benchmark subset
- Human eval on 50 test prompts
- Task-specific accuracy (code, reasoning, facts)

**Deliverable:** `QUANTIZATION_ANALYSIS.md`

---

## Phase 6: Hardware Optimization

### 6A. GPU Allocation Strategies

**Test Configurations:**

1. **Single GPU (RTX 4060 only)**
   - Larger model on 8GB
   - Compare to split approach

2. **Split by task**
   - GPU 0: Fast small model (3B)
   - GPU 1: Slow large model (7B)

3. **Tensor parallelism** (if engine supports mismatched sizes)
   - Custom sharding
   - Load balancing

4. **Pipeline parallelism**
   - Layers split across GPUs
   - Test with TGI/SGLang

**Deliverable:** `GPU_ALLOCATION_STRATEGY.md`

---

### 6B. CPU Offloading

**Hypothesis:** 47GB RAM underutilized, offload to CPU

**Tests:**
1. **Partial offloading**
   - Layers on CPU vs GPU
   - Performance vs memory tradeoff

2. **KV cache on CPU**
   - Large context on CPU memory
   - Speed impact measurement

3. **Hybrid inference**
   - Prefill on GPU
   - Decode on CPU
   - Latency analysis

**Deliverable:** `CPU_OFFLOAD_ANALYSIS.md`

---

### 6C. System-Level Tuning

**OS/Driver Optimizations:**

1. **CUDA optimizations**
   ```bash
   export CUDA_LAUNCH_BLOCKING=0
   export CUDA_VISIBLE_DEVICES=0,1
   export CUDA_DEVICE_ORDER=PCI_BUS_ID
   ```

2. **CPU governor**
   ```bash
   sudo cpupower frequency-set -g performance
   ```

3. **Memory settings**
   ```bash
   # Huge pages for faster memory access
   sudo sysctl -w vm.nr_hugepages=1024
   ```

4. **PCIe optimization**
   - Check PCIe lanes (lspci -vv)
   - Ensure x16 for GPUs

**Deliverable:** `SYSTEM_TUNING_GUIDE.md`

---

## Phase 7: Production Architecture Design

**Input:** Results from Phases 1-6

### Design Decisions

1. **Primary engine selection**
   - Based on benchmark winner
   - Fallback engine choice

2. **Model strategy**
   - Which sizes for which tasks
   - Quantization levels
   - Preload vs on-demand

3. **API design**
   ```
   /v1/chat/completions  (OpenAI-compatible)
   /v1/generate          (Raw generation)
   /v1/embed             (Embeddings)
   /health               (Monitoring)
   /metrics              (Prometheus)
   ```

4. **High availability**
   - Health checks
   - Auto-restart
   - Graceful degradation
   - Cloud fallback triggers

5. **Monitoring**
   - GPU utilization
   - Request latency
   - Queue depth
   - Error rates

**Deliverable:** `PRODUCTION_ARCHITECTURE.md`

---

## Phase 8: Integration with PAI

### 8A. Update PAI Routing Logic

**Current:** `PAI/Tools/Inference.ts`

**New:** Multi-engine dispatcher

```typescript
interface EngineCapability {
  name: string;
  endpoint: string;
  strengths: TaskType[];
  maxContext: number;
  tokensPerSec: number;
  costPerToken: number; // $0 for local
}

const engines: EngineCapability[] = [
  {
    name: "llama.cpp-fast",
    endpoint: "http://localhost:8080",
    strengths: ["classification", "fast_generation"],
    maxContext: 8192,
    tokensPerSec: 45,
    costPerToken: 0
  },
  {
    name: "sglang-structured",
    endpoint: "http://localhost:30000",
    strengths: ["json_extraction", "structured_output"],
    maxContext: 4096,
    tokensPerSec: 38,
    costPerToken: 0
  },
  // ... Claude, Gemini fallbacks
];
```

### 8B. Skill Updates

Update skills to leverage local capabilities:
- OllamaSkill → LocalInferenceSkill
- Route by capability, not by engine
- Automatic fallback to cloud

### 8C. Cost Tracking

Even though local is "free":
- Track GPU hours
- Electricity cost estimation
- Compare to cloud equivalent cost

**Deliverable:** `PAI_INTEGRATION_PLAN.md`

---

## Phase 9: Benchmark Harness

**Create standardized testing framework**

### Test Suite

```typescript
interface BenchmarkTest {
  name: string;
  prompt: string;
  expectedTokens: number;
  taskType: "generation" | "classification" | "json" | "reasoning";
  warmup: boolean;
}

const standardTests: BenchmarkTest[] = [
  {
    name: "fast-classification",
    prompt: "Classify sentiment: 'This is great!' Response: ",
    expectedTokens: 5,
    taskType: "classification",
    warmup: true
  },
  {
    name: "medium-generation",
    prompt: "Write a haiku about coding",
    expectedTokens: 30,
    taskType: "generation",
    warmup: false
  },
  // ... 20+ tests across task types
];
```

### Metrics Collected

For each engine × model × quantization:
1. Time to first token (TTFT)
2. Tokens per second (TPS)
3. GPU memory used
4. System memory used
5. GPU utilization %
6. Power consumption (if measurable)
7. Quality score (vs reference)
8. Concurrent request handling (1, 2, 4, 8 parallel)

### Output Format

```json
{
  "engine": "llama.cpp",
  "model": "qwen2.5-7b-q4_k_m",
  "gpu": "RTX 4060",
  "results": {
    "fast-classification": {
      "ttft_ms": 45,
      "tps": 52.3,
      "memory_mb": 4200,
      "quality_score": 0.95
    }
  }
}
```

**Deliverable:** `BenchmarkHarness.ts` + results database

---

## Timeline Estimate

| Phase | Estimated Time | Priority |
|-------|---------------|----------|
| 1. Ollama Baseline | 2 hours | HIGH |
| 2. llama.cpp Direct | 3 hours | HIGH |
| 3A. LocalAI | 2 hours | MEDIUM |
| 3B. SGLang | 3 hours | HIGH |
| 3C. Aphrodite | 2 hours | LOW |
| 3D. TGI | 2 hours | MEDIUM |
| 3E. MLC LLM | 4 hours | LOW |
| 3F. ExLlamaV2 | 2 hours | MEDIUM |
| 4. Multi-Engine | 4 hours | HIGH |
| 5. Quantization | 6 hours | HIGH |
| 6. Hardware Opt | 4 hours | MEDIUM |
| 7. Architecture | 3 hours | HIGH |
| 8. PAI Integration | 4 hours | HIGH |
| 9. Benchmark Harness | 4 hours | HIGH |
| **TOTAL** | **45 hours** | |

**Aggressive:** 1 week full-time  
**Realistic:** 2-3 weeks part-time  
**Thorough:** 1 month with quality evaluation

---

## Decision Framework

After each phase, decide:

### Stop Conditions (Found Winner)
- Engine is 2x+ faster than Ollama
- Uses 30%+ less memory
- Same or better quality
- Production-stable

### Continue Conditions
- Results inconclusive
- Multiple engines competitive
- Trade-offs need quantification

### Pivot Conditions
- All local options inferior to cloud
- Hardware upgrade more cost-effective
- Hybrid approach optimal

---

## Expected Outcomes

### Likely Scenarios

**Scenario A: llama.cpp Direct Wins**
- Ollama adds overhead
- Replace with llama.cpp server mode
- 20-30% performance gain

**Scenario B: Multi-Engine Optimal**
- llama.cpp for fast queries
- SGLang for structured output
- TGI for batch processing
- Router coordinates

**Scenario C: Specialized by Model Size**
- 3B on GPU 0 for fast tasks
- 7B on GPU 1 for complex tasks
- Route by task complexity

**Scenario D: Hybrid is Best**
- Local for 80% of requests
- Cloud for 20% complex/long-context
- Optimize the 80%, accept cloud cost for 20%

---

## Success Metrics (Final)

After full investigation:

1. **Performance**
   - [ ] 2x throughput increase vs baseline Ollama
   - [ ] <100ms TTFT for simple queries
   - [ ] Support 10+ concurrent requests

2. **Cost**
   - [ ] 90%+ requests on local (vs current 60%)
   - [ ] $200+/month additional savings
   - [ ] ROI on time invested positive

3. **Quality**
   - [ ] No degradation vs baseline
   - [ ] Quantization finds optimal point
   - [ ] Production-stable (99%+ uptime)

4. **Usability**
   - [ ] OpenAI-compatible API
   - [ ] Systemd integration
   - [ ] Monitoring/alerting
   - [ ] Auto-recovery

---

## Risk Mitigation

### If Investigation Fails
- Keep Ollama (baseline works)
- Document what doesn't work
- Re-evaluate when hardware/software changes

### If Time Exceeds Estimate
- Prioritize HIGH items only
- Accept "good enough" over perfect
- Timebox each phase (hard stop at 2x estimate)

### If Nothing Beats Ollama
- Document Ollama as optimal for this hardware
- Focus on tuning Ollama configuration
- Plan for future when hardware upgrades

---

## Next Steps

**Ready to start?**

1. Run Phase 1 (Ollama baseline) - 2 hours
2. Run Phase 2 (llama.cpp direct) - 3 hours  
3. Evaluate and decide on Phase 3 scope

**Or want to adjust the plan first?**

I can:
- Add/remove engines to test
- Change priority order
- Adjust timeline
- Add specific test cases
- Focus on particular aspect (speed vs memory vs quality)

What's your preference?
