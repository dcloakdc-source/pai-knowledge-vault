# System State Before Reboot

**Date:** 2026-06-30 12:00 UTC  
**Reason:** Clear stuck processes (load avg 21-22) before production deployment

---

## Current Issues

### Stuck Processes
```
PID       CPU%  Memory   Uptime    Command
3672698   99.1% 175MB    42+ hours /tmp/llama.cpp/build/bin/llama-server (port 9004)
2479      97.5% 77MB     7+ days   /home/duane/.bun/bin/bun pai-log-scrubber.ts
```

### System Load
- Load average: 21.78, 22.84, 19.91
- Normal expected: <4 on this system
- Impact: 70% performance degradation

---

## Services to Verify After Reboot

### Critical Services
- [ ] Ollama (systemd: ollama.service)
- [ ] Docker containers
- [ ] PostgreSQL
- [ ] Qdrant vector DB

### PAI Services  
- [ ] Check all systemd timers: `systemctl list-timers`
- [ ] Verify no stuck bun processes
- [ ] Confirm GPU accessible: `nvidia-smi`

---

## Models to Verify

### Local Ollama Models
```
qwen2.5:7b        4.7 GB    (primary)
llama3.2:3b       2.0 GB    (fast alternative)
qwen3:8b          5.2 GB    (optional)
```

### Native llama.cpp
```
/tmp/qwen2.5-7b-ollama.gguf  4.4 GB  (copied from Ollama)
```

---

## Post-Reboot Actions

1. **Verify system health**
   ```bash
   uptime  # Should show load <4
   ps aux | awk '$3 > 80'  # Should be empty
   nvidia-smi  # Should show GPUs available
   ```

2. **Start native llama-server**
   ```bash
   CUDA_VISIBLE_DEVICES=1 /tmp/llama.cpp/build/bin/llama-server \
     --model /tmp/qwen2.5-7b-ollama.gguf \
     --host 127.0.0.1 \
     --port 9006 \
     --n-gpu-layers 99 \
     --ctx-size 2048 \
     --parallel 1 \
     --threads 4 \
     --batch-size 512 \
     --metrics > /tmp/llama-gpu1.log 2>&1 &
   ```

3. **Test hybrid router**
   ```bash
   cd ~/.claude/PAI/Tools
   bun HybridInference.ts --prompt "Test" --max-tokens 5
   ```

4. **Verify Ollama**
   ```bash
   curl -s http://localhost:11434/api/tags | jq .
   ollama list
   ```

5. **Re-baseline performance**
   ```bash
   bun BenchmarkLLM.ts --model qwen2.5:7b --output post-reboot-baseline.json
   ```

---

## Configuration Files to Preserve

- `~/.claude/PAI/Tools/HybridInference.ts` ✅
- `~/.claude/PAI/Tools/ValidateHybridQuality.ts` ✅
- `~/.claude/PAI/Tools/TestOllamaConfig.ts` ✅
- `~/.claude/PAI/Tools/LlamaCppLoadBalancer.ts` ✅
- `/tmp/qwen2.5-7b-ollama.gguf` ⚠️ (in /tmp, copy to permanent location)

---

## Copy Model to Permanent Location

**Before reboot:**
```bash
mkdir -p ~/.local/share/llama-models
cp /tmp/qwen2.5-7b-ollama.gguf ~/.local/share/llama-models/
```

**After reboot, update paths in:**
- HybridInference.ts (if needed)
- Native llama-server start command

---

Status: Ready for reboot
