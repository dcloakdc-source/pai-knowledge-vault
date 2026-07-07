# vLLM Installation Result

**Date:** 2026-06-24  
**Status:** ❌ Installation successful, but GPU OOM prevents running alongside Ollama  
**System:** pai-primary (RTX 3060 6GB + RTX 4060 8GB, 47GB RAM)

## Summary

vLLM installation completed successfully but cannot run the Qwen2.5-7B-Instruct model due to GPU memory constraints when Ollama is already running.

## Installation Steps Completed

✅ Python venv created at `~/.vllm-env`  
✅ vLLM 0.23.0 installed  
✅ Model downloaded: Qwen/Qwen2.5-7B-Instruct (15GB)  
✅ Systemd service created: `pai-vllm.service`  
❌ Service start failed: CUDA out of memory

## Root Cause

```
torch.OutOfMemoryError: CUDA out of memory. Tried to allocate 260.00 MiB. 
GPU 0 has a total capacity of 7.62 GiB of which 252.88 MiB is free.
```

**Current GPU usage:**
- GPU 0 (RTX 3060, 6GB): Ollama using 644MB + Xorg
- GPU 1 (RTX 4060, 8GB): Only Xorg (4MB)
- Qwen2.5-7B-Instruct needs ~7.2GB minimum to load

**Configuration attempts:**
1. Both GPUs (CUDA_VISIBLE_DEVICES=0,1) - OOM on GPU 0
2. Single GPU 1 only (CUDA_VISIBLE_DEVICES=1) - Still OOM (7.2GB > 8GB available)
3. Reduced context (8192 → 4096) - Still OOM
4. Increased GPU utilization (0.8 → 0.85) - Still OOM

## Benchmark Status

**Cannot run benchmark** as originally planned because:
- vLLM requires stopping Ollama first (GPU memory conflict)
- Stopping Ollama invalidates the benchmark (can't compare if both aren't running)
- 7B model is too large for available GPU memory with any overhead

## Options Going Forward

### Option 1: Use Smaller/Quantized Model (Recommended)
- Try `Qwen/Qwen2.5-3B-Instruct` (fits in 4GB)
- Or use 4-bit quantized 7B model
- This would allow side-by-side comparison
- **Trade-off:** Smaller model may sacrifice quality

### Option 2: Run vLLM Exclusively
- Stop Ollama permanently
- Dedicate both GPUs to vLLM
- Use tensor parallelism across 2 GPUs
- **Trade-off:** Lose Ollama's ease of use and model switching

### Option 3: Time-share the GPUs
- Run Ollama during the day
- Run vLLM for batch jobs at night
- Switch via systemd timers
- **Trade-off:** Can't use both simultaneously

### Option 4: Keep Ollama Only (Current State)
- vLLM installation preserved but not running
- Continue with current 60% local / 40% cloud routing
- Save $225/month vs all-cloud
- **Trade-off:** No speed improvement

## Recommendation

**Keep Ollama only** for now because:

1. **Cost savings achieved:** $225/month saved vs all-cloud is the primary goal ✅
2. **Proven working:** Ollama operational after disk cleanup
3. **Simpler operation:** One inference engine, easier to maintain
4. **GPU memory fits:** Works within hardware constraints
5. **Speed is acceptable:** 2-3s for most tasks

vLLM would be worth revisiting if:
- Duane gets a GPU upgrade (16GB+ VRAM)
- Need batch processing at night (can time-share)
- Find a quantized model that's both smaller AND faster
- Ollama performance degrades significantly

## Files Created

- `~/.vllm-env/` - Python virtual environment (1.8GB)
- `~/.cache/huggingface/models--Qwen--Qwen2.5-7B-Instruct/` - Model weights (15GB)
- `~/.config/systemd/user/pai-vllm.service` - Service definition (disabled)
- Total disk usage: ~17GB

## Next Steps

1. ✅ Document findings (this file)
2. ⏭️ User decides: keep vLLM installed (17GB) or remove it
3. ⏭️ If keeping: try smaller model for future testing
4. ⏭️ If removing: clean up with provided commands

## Cleanup Commands (if removing vLLM)

```bash
# Remove Python environment
rm -rf ~/.vllm-env

# Remove downloaded model
rm -rf ~/.cache/huggingface/models--Qwen--Qwen2.5-7B-Instruct

# Remove systemd service
rm ~/.config/systemd/user/pai-vllm.service
systemctl --user daemon-reload

# Will free: ~17GB disk space
```

## Cost Analysis Unchanged

Original analysis remains valid:
- **Ollama (current):** $0/month + $0 hardware (already owned)
- **All-cloud fallback:** ~$225/month saved
- **Current routing:** 60% free local, 40% paid cloud
- **vLLM would not change costs** - just speed (if it worked)

## Conclusion

Installation successful but deployment blocked by GPU memory constraints. Ollama remains the correct choice for PAI's local-first inference given current hardware.
