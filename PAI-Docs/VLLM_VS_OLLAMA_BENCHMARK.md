# vLLM vs Ollama Benchmark Guide

**Purpose:** Install vLLM alongside Ollama and run comprehensive performance comparison  
**Target System:** pai-primary (Ubuntu, 2x NVIDIA GPUs, 47GB RAM)  
**Date:** 2026-06-24

---

## Quick Start

```bash
# 1. Install vLLM
bash ~/.claude/PAI/Tools/vllm-install.sh

# 2. Start vLLM service
systemctl --user start pai-vllm.service

# 3. Run benchmark
bash ~/.claude/PAI/Tools/vllm-benchmark.sh

# 4. View results
cat ~/MEMORY/STATE.claude/vllm-benchmark-*.md
```

---

## What Gets Installed

### vLLM Installation

1. **Python virtual environment** at `~/.vllm-env`
   - Isolated from system Python
   - Contains vLLM + dependencies (~2GB)

2. **Qwen2.5-7B-Instruct model** in `~/.cache/huggingface/`
   - Same model family as Ollama's qwen3:8b
   - ~15GB download
   - Apples-to-apples comparison

3. **systemd service** `pai-vllm.service`
   - Runs on port 8001 (Ollama is 11434)
   - OpenAI-compatible API
   - Auto-restart on failure

**Total disk usage:** ~17GB (2GB packages + 15GB model)

---

## Pre-Installation Checklist

### System Requirements

| Requirement | Minimum | Recommended | Your System |
|-------------|---------|-------------|-------------|
| **Python** | 3.8+ | 3.10+ | ✅ 3.12.3 |
| **CUDA** | 11.0+ | 12.0+ | ✅ Installed (nvcc present) |
| **GPU** | 1x 6GB VRAM | 2x 8GB+ | ✅ RTX 3060 (6GB) + RTX 4060 (8GB) |
| **RAM** | 16GB | 32GB+ | ✅ 47GB |
| **Disk free** | 15GB | 30GB+ | ✅ 43GB |
| **OS** | Ubuntu 20.04+ | Ubuntu 22.04+ | ✅ Ubuntu |

**Verdict:** Your system meets all requirements ✅

### Disk Space Check

**Current:** 92% full (43GB free)  
**Need:** 17GB for vLLM + model  
**After install:** ~84% full (26GB free)  

**Recommendation:** Proceed, but monitor disk closely during install.

---

## Installation Steps

### Step 1: Run Installation Script

```bash
bash ~/.claude/PAI/Tools/vllm-install.sh
```

**What it does:**
1. Checks requirements (Python, CUDA, disk, RAM)
2. Creates Python virtual environment
3. Installs vLLM package (~5-10 minutes)
4. Downloads Qwen2.5-7B model (~10-15 minutes, 15GB)
5. Creates systemd service file

**Prompts:**
- "Install vLLM?" → Type `y`
- "Download model?" → Type `y`

**Expected duration:** 15-25 minutes (depends on internet speed)

### Step 2: Verify Installation

```bash
# Check vLLM installed
source ~/.vllm-env/bin/activate
python -c "import vllm; print(vllm.__version__)"
# Should print: 0.x.x

# Check model downloaded
ls -lh ~/.cache/huggingface/hub/models--Qwen--Qwen2.5-7B-Instruct
# Should show ~15GB of files

# Check service file exists
cat ~/.config/systemd/user/pai-vllm.service
```

### Step 3: Start vLLM Service

```bash
# Reload systemd
systemctl --user daemon-reload

# Start service
systemctl --user start pai-vllm.service

# Check status
systemctl --user status pai-vllm.service
```

**Expected output:**
```
● pai-vllm.service - PAI vLLM Server
     Active: active (running) since ...
     Memory: 8-12GB
```

**First startup takes 30-60 seconds** as model loads into GPU memory.

### Step 4: Test vLLM API

```bash
# Check models endpoint
curl http://localhost:8001/v1/models

# Test inference
curl http://localhost:8001/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5-7b",
    "messages": [{"role": "user", "content": "What is 2+2?"}]
  }'
```

**Expected:** JSON response with "4" or similar.

---

## Benchmark Execution

### Run Full Benchmark Suite

```bash
bash ~/.claude/PAI/Tools/vllm-benchmark.sh
```

**What gets tested:**

1. **Latency Tests** (6 task types, single request each)
   - Simple math
   - Classification
   - Summarization
   - Reasoning
   - Code generation
   - JSON extraction

2. **Throughput Test** (10 concurrent requests)
   - Measures req/sec under load
   - Tests concurrency handling

**Duration:** ~2-3 minutes

### Sample Output

```
[INFO] PAI vLLM vs Ollama Benchmark
====================================

[INFO] Checking service availability...
[INFO] ✓ Ollama is running (192.168.50.20:11434)
[INFO] ✓ vLLM is running (localhost:8001)

[INFO] Running benchmarks...

[INFO] Testing: simple_math
[INFO] Prompt: What is 15% of 340? Just give me the number.
  Ollama: ✓ 3247ms - 51
  vLLM:   ✓ 892ms - 51
  → vLLM 3.64x faster

[INFO] Testing: classification
[INFO] Prompt: Classify the sentiment...
  Ollama: ✓ 2156ms - Positive
  vLLM:   ✓ 541ms - Positive
  → vLLM 3.99x faster

...

[RESULT] Throughput (10 concurrent requests):
[RESULT]   Ollama: 18542ms total, 0.54 req/sec
[RESULT]   vLLM:   4231ms total, 2.36 req/sec
```

### Results File

**Location:** `~/MEMORY/STATE.claude/vllm-benchmark-YYYYMMDD-HHMMSS.jsonl`

**Format:** JSON Lines (one JSON object per line)

**Markdown report:** Same location, `.md` extension

---

## Expected Benchmark Results

### Predicted Performance (Based on Literature)

| Metric | Ollama | vLLM | Expected Winner |
|--------|--------|------|----------------|
| **Latency (single request)** | 2-5s | 0.5-1.5s | vLLM (3-4x faster) |
| **Throughput (concurrent)** | 0.5 req/sec | 2-5 req/sec | vLLM (4-10x faster) |
| **First token latency** | 2-3s | 0.3-0.8s | vLLM (3-5x faster) |
| **Memory efficiency** | Good | Excellent | vLLM |
| **GPU utilization** | 50-70% | 80-95% | vLLM |

### Why vLLM Should Win

1. **PagedAttention** — More efficient memory management
2. **Continuous batching** — Handles concurrent requests better
3. **Optimized kernels** — Custom CUDA kernels for operations
4. **Less overhead** — Lighter runtime than Ollama's abstraction layer

### Scenarios Where Ollama Might Win

1. **First request** — If model already loaded, Ollama may be faster (vLLM cold start)
2. **Very simple tasks** — Overhead may dominate (unlikely)
3. **Setup issues** — If vLLM misconfigured (GPU not detected, wrong CUDA version)

---

## Interpreting Results

### What to Look For

#### 1. Latency Improvement

```
Ollama: 3000ms
vLLM:   800ms
Speedup: 3.75x
```

**Good:** 2-4x faster is expected  
**Great:** 4-6x faster means vLLM is well-optimized  
**Bad:** <2x or slower means configuration issue  

#### 2. Throughput Improvement

```
Ollama: 0.5 req/sec
vLLM:   2.5 req/sec
Speedup: 5x
```

**Good:** 3-5x throughput is expected  
**Great:** 5-10x means vLLM's batching is working  
**Bad:** <2x means not enough concurrent load to show benefit  

#### 3. Quality Comparison

**Check:** Do both give correct answers?

```bash
# Compare outputs in results file
jq 'select(.task=="simple_math")' ~/MEMORY/STATE.claude/vllm-benchmark-*.jsonl
```

**Expected:** Both should give same/similar answers (both Qwen 2.5 family)

#### 4. Resource Usage

```bash
# Check GPU memory during benchmark
ssh pai-primary "nvidia-smi"
```

**Ollama:** 4-6GB VRAM  
**vLLM:** 6-8GB VRAM (uses more, but more efficiently)  

---

## Decision Matrix

### After Benchmarking, Choose Based On:

| Scenario | Stick with Ollama | Switch to vLLM |
|----------|------------------|----------------|
| **vLLM is 2-3x faster** | ✅ If ease > speed | ⚠️ If speed matters |
| **vLLM is 4-6x faster** | ⚠️ Hard to justify | ✅ Clear winner |
| **vLLM is slower** | ✅ Obvious | ❌ Config issue |
| **High concurrent load (>5 req/sec)** | ❌ Ollama struggles | ✅ vLLM excels |
| **Single-user, low load** | ✅ Ollama fine | ⚠️ vLLM overkill |
| **Setup complexity matters** | ✅ Ollama simpler | ❌ vLLM more work |

### Cost-Benefit Analysis

**Ollama:**
- ✅ Already working
- ✅ Zero setup time
- ✅ Easy to manage
- ❌ Slower (if benchmarks confirm)

**vLLM:**
- ✅ 3-6x faster (expected)
- ✅ Better under load
- ❌ 1-2 hours setup
- ❌ More complex (Python env, systemd service)

**Break-even:** If you save >2 hours/month from faster inference, vLLM pays for setup time.

---

## Integration with PAI

### Adding vLLM to Inference.ts

```typescript
// PAI/Tools/Inference.ts

const VLLM_HOST = "http://localhost:8001";

async function inferenceVLLM(
  options: InferenceOptions,
  modelName: string
): Promise<InferenceResult> {
  const startTime = Date.now();
  
  const response = await fetch(`${VLLM_HOST}/v1/chat/completions`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      model: modelName,
      messages: [
        { role: 'system', content: options.systemPrompt },
        { role: 'user', content: options.userPrompt },
      ],
      max_tokens: options.options?.max_tokens || 4000,
      temperature: options.options?.temperature || 0.7,
    }),
    signal: options.timeout ? AbortSignal.timeout(options.timeout) : undefined,
  });
  
  const result = await response.json();
  const latencyMs = Date.now() - startTime;
  const output = result.choices[0]?.message?.content || '';
  
  return {
    success: true,
    output,
    latencyMs,
    model: modelName,
  };
}
```

### Updated Routing Matrix

```json
{
  "routing_matrix": {
    "fast_classification": {
      "primary": "vllm/qwen2.5-7b",    // NEW: Use vLLM
      "fallback": "ollama/qwen3:8b",   // Keep Ollama as backup
      "cloud_fallback": "claude/haiku"
    },
    "complex_reasoning": {
      "primary": "vllm/qwen2.5-7b",    // NEW
      "fallback": "ollama/gemma2:9b",
      "cloud_fallback": "claude/opus"
    }
  },
  "vllm_enabled": true,
  "vllm_host": "http://localhost:8001"
}
```

### Three-Tier Routing

```
Request arrives
  ↓
Try vLLM (fastest, free)
  ↓ (if down/timeout)
Try Ollama (slower, free)
  ↓ (if down/timeout)
Claude (guaranteed, paid)
```

---

## Troubleshooting

### vLLM Won't Start

**Error:** `ModuleNotFoundError: No module named 'vllm'`

```bash
# Solution: Activate virtual environment
source ~/.vllm-env/bin/activate
python -c "import vllm"
```

**Error:** `CUDA out of memory`

```bash
# Solution: Reduce GPU memory utilization
# Edit ~/.config/systemd/user/pai-vllm.service
# Change: --gpu-memory-utilization 0.9
# To:     --gpu-memory-utilization 0.7
systemctl --user daemon-reload
systemctl --user restart pai-vllm.service
```

**Error:** `Model not found: qwen2.5-7b`

```bash
# Solution: Check model downloaded
ls ~/.cache/huggingface/hub/models--Qwen--Qwen2.5-7B-Instruct

# Re-download if missing
source ~/.vllm-env/bin/activate
python -c "
from huggingface_hub import snapshot_download
snapshot_download('Qwen/Qwen2.5-7B-Instruct')
"
```

### Benchmark Fails

**Error:** `Ollama is not responding`

```bash
# Solution: Start Ollama
ssh pai-primary "systemctl restart ollama.service"
```

**Error:** `vLLM is not responding`

```bash
# Solution: Check vLLM status
systemctl --user status pai-vllm.service

# Check logs
journalctl --user -u pai-vllm.service -n 50
```

### Slow Performance

**Issue:** vLLM not faster than Ollama

```bash
# Check GPU usage
ssh pai-primary "nvidia-smi"

# Expected: 1-2 GPUs at 80%+ during inference
# If 0%: Model not using GPU

# Solution: Check CUDA_VISIBLE_DEVICES in service file
cat ~/.config/systemd/user/pai-vllm.service | grep CUDA

# Should be: Environment=CUDA_VISIBLE_DEVICES=0,1
```

---

## Cleanup / Uninstall

### If You Decide Not to Use vLLM

```bash
# 1. Stop and disable service
systemctl --user stop pai-vllm.service
systemctl --user disable pai-vllm.service

# 2. Remove service file
rm ~/.config/systemd/user/pai-vllm.service
systemctl --user daemon-reload

# 3. Remove Python environment
rm -rf ~/.vllm-env

# 4. Remove model (optional, frees 15GB)
rm -rf ~/.cache/huggingface/hub/models--Qwen--Qwen2.5-7B-Instruct

# 5. Verify disk space recovered
df -h /
```

**Disk freed:** ~17GB

---

## Next Steps After Benchmarking

### If vLLM is Significantly Faster (4x+)

1. ✅ Keep vLLM running
2. ✅ Update `inference-matrix.json` to use vLLM as primary
3. ✅ Run for 1 week, monitor stability
4. ✅ Measure actual cost savings from reduced latency
5. ✅ Document findings

### If vLLM is Marginally Faster (2-3x)

1. ⚠️ Keep both running for 1 week
2. ⚠️ A/B test on real workload
3. ⚠️ Measure impact on productivity
4. ⚠️ Decide based on data

### If vLLM is Slower or Equal

1. ❌ Investigate configuration (GPU not detected?)
2. ❌ Check CUDA version compatibility
3. ❌ Re-run benchmark
4. ❌ If still slow, uninstall vLLM

---

## Reference Commands

```bash
# Start vLLM
systemctl --user start pai-vllm.service

# Stop vLLM
systemctl --user stop pai-vllm.service

# Check status
systemctl --user status pai-vllm.service

# View logs
journalctl --user -u pai-vllm.service -f

# Test API
curl http://localhost:8001/v1/models

# Run benchmark
bash ~/.claude/PAI/Tools/vllm-benchmark.sh

# View results
ls -lh ~/MEMORY/STATE.claude/vllm-benchmark-*.md
cat ~/MEMORY/STATE.claude/vllm-benchmark-*.md

# Check disk usage
df -h /
du -sh ~/.vllm-env ~/.cache/huggingface
```

---

**Related Docs:**
- `PAI/DOCUMENTATION/OLLAMA_CAPABILITY_ASSESSMENT.md` — Ollama quality benchmarks
- `PAI/DOCUMENTATION/OLLAMA_TIMEOUT_INVESTIGATION.md` — Disk space troubleshooting
- `PAI/DOCUMENTATION/OPENROUTER_VS_OPENCODE.md` — Alternative architectures
- `PAI/Tools/vllm-install.sh` — Installation automation
- `PAI/Tools/vllm-benchmark.sh` — Benchmark automation

**Installation artifacts:**
- Virtual environment: `~/.vllm-env/`
- Model cache: `~/.cache/huggingface/`
- Service file: `~/.config/systemd/user/pai-vllm.service`
- Benchmark results: `~/MEMORY/STATE.claude/vllm-benchmark-*.{jsonl,md}`
