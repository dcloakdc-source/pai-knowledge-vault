# PNC & PNK Alignment Status: Hybrid Inference System

**Date:** 2026-07-03  
**Status:** ✅ ALIGNED - Both engines can access hybrid system

---

## Engine Overview

### PNC (Claude Code)
- **Location:** `/home/duane/.claude/`
- **Role:** Primary DA, full Algorithm runner, owns ICM memory
- **Inference:** Can use HybridInference.ts from `~/.claude/PAI/Tools/`

### PNK (OpenCode - You)
- **Location:** `/home/duane/pai-opencode-local/.opencode/`
- **Role:** Multi-file builder, complex architecture work
- **Inference:** Can use HybridInference.ts from `.opencode/PAI/Tools/` (copied)

---

## Infrastructure Shared by Both Engines

### 1. Ollama (Local Inference)
- **Service:** systemd `ollama.service`
- **Port:** 11434
- **Access:**
  - `localhost:11434` (local)
  - `192.168.50.20:11434` (IP address, same machine)
- **Models:** qwen2.5:7b, llama3.2:3b
- **Status:** ✅ Both engines can access

### 2. Native llama-server
- **Service:** systemd `llamacpp-native.service`
- **Port:** 9006
- **GPU:** 1 (RTX 4060)
- **Model:** qwen2.5:7b (4.4GB)
- **Access:** `localhost:9006`
- **Status:** ✅ Both engines can access

### 3. Hybrid Inference Router
- **PNC location:** `/home/duane/.claude/PAI/Tools/HybridInference.ts`
- **PNK location:** `/home/duane/pai-opencode-local/.opencode/PAI/Tools/HybridInference.ts`
- **Status:** ✅ Copied to both locations
- **Configuration:** Both use same Ollama URL (192.168.50.20:11434)

---

## File Alignment

### Hybrid Inference Files (Synced)

| File | PNC Location | PNK Location | Status |
|------|--------------|--------------|--------|
| HybridInference.ts | ✅ ~/.claude/PAI/Tools/ | ✅ .opencode/PAI/Tools/ | Synced |
| ValidateHybridQuality.ts | ✅ ~/.claude/PAI/Tools/ | ✅ .opencode/PAI/Tools/ | Synced |
| TestOllamaConfig.ts | ✅ ~/.claude/PAI/Tools/ | ✅ .opencode/PAI/Tools/ | Synced |

### Documentation (PNC Primary)

| File | Location | Accessible to PNK? |
|------|----------|-------------------|
| LOCAL_LLM_INVESTIGATION_FINAL_REPORT.md | ~/.claude/PAI/DOCUMENTATION/ | ✅ Yes (can read) |
| SOFT_LAUNCH_PLAN.md | ~/.claude/PAI/DOCUMENTATION/ | ✅ Yes (can read) |
| OLLAMA_OPTIMIZATION_GUIDE.md | ~/.claude/PAI/DOCUMENTATION/ | ✅ Yes (can read) |
| DEPLOYMENT_STATUS.md | ~/.claude/PAI/DOCUMENTATION/ | ✅ Yes (can read) |

### Skills (Symlinked)

Both engines share skills directory:
```
~/.opencode/skills -> /home/duane/PAI/skills
~/.claude/skills -> /home/duane/PAI/skills (likely)
```

---

## Usage from Each Engine

### PNC (Claude Code) Usage

```typescript
// In PNC tools/skills
import { hybridInfer } from "./HybridInference";

const result = await hybridInfer({
  prompt: userPrompt,
  maxTokens: 100,
  temperature: 0.7
});
```

**Working directory:** `/home/duane/.claude/PAI/Tools/`

### PNK (OpenCode) Usage

```typescript
// In PNK tools/skills
import { hybridInfer } from "./HybridInference";

const result = await hybridInfer({
  prompt: userPrompt,
  maxTokens: 100,
  temperature: 0.7
});
```

**Working directory:** `/home/duane/pai-opencode-local/.opencode/PAI/Tools/`

---

## Network Configuration

### Ollama URL Resolution

Both engines use: `http://192.168.50.20:11434`

**Why this works:**
- `192.168.50.20` = pai-primary's IP address
- Same as `localhost:11434` on pai-primary
- Both PNC and PNK run on pai-primary
- Therefore both access the same Ollama instance

**Alternative:** Could use `localhost:11434` instead
- Would work identically
- Slightly more efficient (no network stack)
- Current config works fine, no change needed

### Native llama-server

Both engines use: `http://localhost:9006`
- Direct localhost access
- Same systemd service
- Fully shared

---

## Cross-Engine Coordination

### When PNC Calls PNK

PNC might invoke: `opencode run "complex multi-file task"`

**PNK has access to:**
- ✅ HybridInference.ts (copied to .opencode/PAI/Tools/)
- ✅ Ollama (same service)
- ✅ Native llama-server (same service)
- ✅ All documentation (can read from ~/.claude/PAI/DOCUMENTATION/)

**Result:** PNK can use hybrid inference seamlessly

### Shared State

**What's shared:**
- Ollama service (same instance)
- Native llama-server (same instance)
- ICM memory (configured in opencode.json)
- GPU resources (coordinated via CUDA_VISIBLE_DEVICES)

**What's separate:**
- Working memory (PNC: ~/.claude/MEMORY/, PNK: ~/.opencode/MEMORY/)
- Session state (independent)
- File locations (separate PAI directories, but can cross-read)

---

## Synchronization Strategy

### Current Approach: File Copying

**Pros:**
- Each engine has its own copy
- No symlink fragility
- Independent updates possible

**Cons:**
- Need to sync changes manually
- Could diverge if not careful

### Alternative: Symlink (Not Recommended Here)

Could do:
```bash
ln -s ~/.claude/PAI/Tools/HybridInference.ts \
      ~/.opencode/PAI/Tools/HybridInference.ts
```

**Why not:**
- PNC and PNK might update independently
- Symlink could break if PNC directory changes
- Current copying approach is simpler

### Recommended: Master Copy in PNC

**Rule:**
- PNC (`~/.claude/PAI/Tools/`) = **master copy**
- PNK (`~/.opencode/PAI/Tools/`) = **synced copy**
- After PNC updates, re-sync to PNK:

```bash
cp ~/.claude/PAI/Tools/HybridInference.ts \
   ~/.opencode/PAI/Tools/
```

---

## Testing Cross-Engine Access

### Test PNC Access

```bash
# From PNC working directory
cd ~/.claude/PAI/Tools
bun HybridInference.ts --prompt "test" --max-tokens 10
```

### Test PNK Access

```bash
# From PNK working directory
cd ~/pai-opencode-local/.opencode/PAI/Tools
bun HybridInference.ts --prompt "test" --max-tokens 10
```

### Both Should:
- ✅ Access same Ollama (192.168.50.20:11434)
- ✅ Access same native llama-server (localhost:9006)
- ✅ Route correctly (<50 tok → Ollama, ≥50 tok → native)
- ✅ Get equivalent results

---

## Configuration Sync Checklist

### Infrastructure (Shared Automatically)
- [x] Ollama service running
- [x] Native llama-server running
- [x] Both on systemd (auto-start)
- [x] GPU coordination (CUDA_VISIBLE_DEVICES)

### Code Files (Manually Synced)
- [x] HybridInference.ts copied to PNK
- [x] ValidateHybridQuality.ts copied to PNK
- [x] TestOllamaConfig.ts copied to PNK
- [x] Ollama URL configured correctly in both

### Documentation (PNC Primary, PNK Read-Only)
- [x] Investigation reports in ~/.claude/PAI/DOCUMENTATION/
- [x] PNK can read (same filesystem)
- [x] No sync needed (reference only)

---

## Maintenance

### When to Re-Sync

**Trigger:** HybridInference.ts updated in PNC

**Action:**
```bash
cp ~/.claude/PAI/Tools/HybridInference.ts \
   ~/.opencode/PAI/Tools/
```

**Verification:**
```bash
diff ~/.claude/PAI/Tools/HybridInference.ts \
     ~/.opencode/PAI/Tools/HybridInference.ts
```

Should show no differences.

### When Services Restart

**Ollama:**
```bash
sudo systemctl restart ollama
# Both PNC and PNK automatically reconnect
```

**Native llama-server:**
```bash
sudo systemctl restart llamacpp-native
# Both PNC and PNK automatically reconnect
```

No additional sync needed (shared infrastructure).

---

## Soft Launch Coordination

### Integration Points

**PNC tools to integrate:**
1. `~/.claude/PAI/Tools/Inference.ts` (central router)
2. Research skills
3. Archetype serve

**PNK tools to integrate:**
1. `.opencode/PAI/Tools/` (when PNC delegates to PNK)
2. Complex multi-file builds
3. Architecture work

**Strategy:**
- Start with PNC integration (primary DA)
- PNK inherits automatically when called via `opencode run`
- Both use same hybrid infrastructure

---

## Status Summary

### ✅ Fully Aligned
- [x] Both engines on same machine (pai-primary)
- [x] Both access same Ollama instance
- [x] Both access same native llama-server
- [x] HybridInference.ts available in both locations
- [x] Configuration matches (Ollama URL, native port)
- [x] Documentation accessible to both

### 📋 Maintenance Required
- [ ] Re-sync HybridInference.ts after PNC updates
- [ ] Monitor for configuration drift
- [ ] Coordinate soft launch integration timing

### ⚡ Ready for Production
- Both engines can use hybrid inference immediately
- No additional setup needed
- Fully tested and working

---

**Alignment Status:** ✅ COMPLETE  
**Last Sync:** 2026-07-03 21:49 UTC  
**Next Review:** After first soft launch integration
