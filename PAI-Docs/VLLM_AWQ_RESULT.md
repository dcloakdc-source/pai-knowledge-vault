# vLLM AWQ Quantized Model - Installation Result

**Date:** 2026-06-24  
**Status:** ❌ Installation successful, runtime crashes due to FlashInfer compilation failures  
**System:** pai-primary (RTX 3060 6GB + RTX 4060 8GB, 47GB RAM)

## Summary

AWQ quantized model (5.2GB) downloaded successfully and fits in GPU memory, but vLLM 0.23.0 has runtime compilation issues with FlashInfer that prevent the service from starting.

## What Worked

✅ AutoAWQ 0.2.9 installed (deprecated but functional)  
✅ Qwen/Qwen2.5-7B-Instruct-AWQ downloaded (5.2GB)  
✅ Model loads into GPU memory (5.7GB used on RTX 4060)  
✅ No OOM errors (unlike FP16 version)

## What Failed

❌ FlashInfer JIT compilation fails with ninja build errors  
❌ Service crashes in a restart loop (12+ restarts)  
❌ Cannot disable FlashInfer (--disable-flashinfer flag doesn't exist in v0.23.0)  
❌ Environment variable VLLM_ATTENTION_BACKEND=FLASH_ATTN doesn't prevent FlashInfer usage

## Root Cause Error

```
subprocess.CalledProcessError: Command '['ninja', '-v', '-C', 
'/home/duane/.cache/flashinfer/0.6.12/89/cached_ops/sampling', 
'-f', '/home/duane/.cache/flashinfer/0.6.12/89/cached_ops/sampling/build.ninja']' 
returned non-zero exit status 1.
```

FlashInfer (v0.6.12) tries to JIT-compile CUDA kernels at runtime but fails. This is likely due to:
1. CUDA toolkit version mismatch  
2. Missing build dependencies  
3. Insufficient GPU compute capability detection  
4. Known bug in vLLM 0.23.0 + FlashInfer 0.6.12

## Attempts Made

| Attempt | Configuration | Result |
|---------|--------------|--------|
| 1 | FP16 model, 8192 context, both GPUs | OOM (7.2GB > 8GB) |
| 2 | FP16 model, 4096 context, GPU 1 only | OOM (still >8GB) |
| 3 | AWQ model, 8192 context, 0.90 GPU util | FlashInfer crash |
| 4 | AWQ model, 4096 context, --disable-flashinfer | Flag doesn't exist |
| 5 | AWQ model, 4096 context, VLLM_ATTENTION_BACKEND=FLASH_ATTN | Still tries FlashInfer |
| 6 | AWQ model, 4096 context, 0.80 GPU util | Still crashes |

## Files Created

- `~/.vllm-env/` - Python venv with vLLM 0.23.0 + AutoAWQ (2GB)
- `~/.cache/huggingface/models--Qwen--Qwen2.5-7B-Instruct-AWQ/` - Model (5.2GB)
- `~/.cache/huggingface/models--Qwen--Qwen2.5-7B-Instruct/` - FP16 model (15GB, unused)
- `~/.cache/flashinfer/` - Failed compilation artifacts (~500MB)
- `~/.config/systemd/user/pai-vllm.service` - Service file (disabled)
- **Total disk usage:** ~23GB

## Why vLLM Failed (Technical)

vLLM 0.23.0 has aggressive JIT compilation for performance:
1. **torch.compile** - compiles model graph (worked, took 19s)
2. **FlashInfer autotune** - compiles optimized CUDA kernels (failed)

The FlashInfer step is mandatory in vLLM 0.23.0 for attention operations and cannot be disabled without source code modification.

## Potential Fixes (Not Attempted - Too Complex)

### Fix 1: Downgrade vLLM
```bash
pip install vllm==0.21.0  # older version without FlashInfer
```
**Risk:** May not support AWQ quantization properly

### Fix 2: Install Full CUDA Toolkit
```bash
sudo apt install nvidia-cuda-toolkit-12-2
```
**Risk:** 6GB download, may conflict with driver CUDA

### Fix 3: Pre-compile FlashInfer
```bash
pip install flashinfer --no-binary flashinfer
```
**Risk:** Requires C++ compiler, may still fail

### Fix 4: Use vLLM Docker Image
```bash
docker run --gpus all vllm/vllm-openai:latest
```
**Risk:** Different environment, harder to integrate with systemd

## Recommendation

**Abandon vLLM for now. Keep Ollama.**

### Reasons:
1. **vLLM is too brittle** - JIT compilation failures, version sensitivity
2. **Ollama works** - proven operational after disk cleanup
3. **Cost savings achieved** - $225/month saved vs all-cloud (primary goal ✅)
4. **Speed acceptable** - 2-3s for most tasks
5. **Simplicity wins** - one inference engine, easier to maintain
6. **Time investment** - 2+ hours debugging vLLM, no ROI yet

### When to Revisit vLLM:
- vLLM releases stable version (0.24+) with better error handling
- Get GPU upgrade (16GB+ VRAM) for FP16 models
- Find pre-compiled vLLM binaries that work
- Ollama performance degrades significantly

## Alternative: Try Ollama Performance Tuning

Instead of vLLM, optimize what we have:

1. **Use smaller models for simple tasks**
   ```bash
   ollama run llama3.2:3b  # faster for classification
   ```

2. **Tune Ollama settings**
   ```bash
   export OLLAMA_NUM_PARALLEL=4
   export OLLAMA_MAX_LOADED_MODELS=2
   ```

3. **Monitor and profile**
   ```bash
   ollama ps  # check what's loaded
   ```

This is pragmatic engineering: optimize the working system rather than chase a broken one.

## Cleanup Commands

If removing vLLM entirely:

```bash
# Remove Python environment
rm -rf ~/.vllm-env

# Remove both models (FP16 + AWQ)
rm -rf ~/.cache/huggingface/models--Qwen--Qwen2.5-7B-Instruct-AWQ
rm -rf ~/.cache/huggingface/models--Qwen--Qwen2.5-7B-Instruct

# Remove failed compilation cache
rm -rf ~/.cache/flashinfer
rm -rf ~/.cache/vllm

# Remove systemd service
rm ~/.config/systemd/user/pai-vllm.service
systemctl --user daemon-reload

# Will free: ~23GB disk space
```

## Conclusion

vLLM AWQ installation was technically feasible (model fits in GPU) but operationally blocked by FlashInfer runtime compilation failures. After 6 configuration attempts and 2+ hours of debugging, the pragmatic choice is to keep Ollama as the local inference engine.

**Bottom line:** Working Ollama saving $225/month > Broken vLLM saving nothing.
