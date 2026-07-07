# Complete PNC ↔ PNK Alignment Status

**Date:** 2026-07-03  
**Investigation:** Hybrid Inference System Cross-Engine Deployment

---

## Executive Summary

### ✅ Fully Aligned (Shared Infrastructure)
- **Skills:** 128 skills (100% - symlinked to shared directory)
- **Ollama:** Running and accessible to both engines
- **Native llama-server:** Running and accessible to both engines
- **ICM/Qdrant:** Running and accessible to both engines
- **Hybrid inference files:** All 3 core files present in both locations

### ⚠️ Partially Aligned
- **Hooks:** 4% aligned (5 of 112 hooks in PNK)
- **PAI/Tools:** 24% aligned (74 of 308 tools in PNK)

### 🎯 Design Intent
PNK is **intentionally lighter** than PNC:
- PNC = Primary DA, full Algorithm runner, all hooks/tools
- PNK = Builder for complex multi-file work delegated by PNC
- **By design:** PNK doesn't need all PNC hooks/tools

---

## Detailed Alignment Report

### 1. Skills Directory ✅ 100% ALIGNED

```
PNC: ~/.claude/skills/ (128 skills)
PNK: ~/.opencode/skills/ -> /home/duane/PAI/skills (SYMLINK)
```

**Status:** ✅ **Perfectly aligned** - both engines use the same physical directory

**Skills available to both:**
- All 128 skills including HybridInference integration points
- Agents skill (for agent composition)
- Research skills (for inference integration)
- All skill metadata synced automatically

**Action:** ✅ No action needed - already optimal

---

### 2. Infrastructure Services ✅ FULLY SHARED

| Service | Port | Status | PNC Access | PNK Access |
|---------|------|--------|------------|------------|
| **Ollama** | 11434 | ✅ Running | ✅ Yes | ✅ Yes |
| **Native llama-server** | 9006 | ✅ Running | ✅ Yes | ✅ Yes |
| **ICM (Qdrant)** | 6333 | ✅ Running | ✅ Yes | ✅ Yes |

**Configuration:**
- Both engines use `localhost:11434` for Ollama
- Both engines use `localhost:9006` for native llama-server
- Both engines use `localhost:6333` for ICM/Qdrant

**Action:** ✅ No action needed - fully shared

---

### 3. Hybrid Inference Files ✅ SYNCED

| File | PNC | PNK | Status |
|------|-----|-----|--------|
| **HybridInference.ts** | ✅ | ✅ | Synced 2026-07-03 |
| **ValidateHybridQuality.ts** | ✅ | ✅ | Synced 2026-07-03 |
| **TestOllamaConfig.ts** | ✅ | ✅ | Synced 2026-07-03 |

**Locations:**
- PNC: `~/.claude/PAI/Tools/`
- PNK: `~/.opencode/PAI/Tools/`

**Content:** Identical (copied 2026-07-03 21:49 UTC)

**Action:** ✅ Core files synced for hybrid inference deployment

---

### 4. PAI/Tools Directory ⚠️ 24% ALIGNED

```
PNC: 308 .ts files in ~/.claude/PAI/Tools/
PNK: 74 .ts files in ~/.opencode/PAI/Tools/
```

**Analysis:**

**Critical tools present in both:**
- ✅ HybridInference.ts
- ✅ ValidateHybridQuality.ts  
- ✅ TestOllamaConfig.ts
- ✅ Inference.ts
- ✅ Core utilities

**Tools only in PNC (234 files):**
- Algorithm-specific tools (AlgorithmReplay, AlgorithmTracker, etc.)
- Agent coordination (AgentLogger, AgentMessenger, etc.)
- Specialized analyzers (many one-off scripts)
- Development/debugging tools

**Design rationale:**
- PNK is a **builder**, not a full DA
- PNC delegates to PNK for specific multi-file tasks
- PNK doesn't run Algorithm itself → doesn't need Algorithm tools
- PNK doesn't coordinate agents → doesn't need agent tools

**Action:** ⚠️ **Selective sync recommended**

---

### 5. Hooks Directory ⚠️ 4% ALIGNED

```
PNC: 112 .hook.ts files in ~/.claude/hooks/
PNK: 5 .hook.ts files in ~/.opencode/hooks/
```

**Hooks present in both:**
- CheckpointPerISC.hook.ts
- PostCompactRestore.hook.ts
- PromptProcessing.hook.ts
- SessionStart.hook.ts
- FileChanged.hook.ts

**Hooks only in PNC (107):**
- Identity validation hooks
- Algorithm tracking hooks
- Security/permission hooks
- Budget tracking hooks
- Activity logging hooks

**Design rationale:**
- PNC = primary DA with full lifecycle hooks
- PNK = specialized builder invoked by PNC
- PNK has minimal hooks for its narrow scope
- Most PNC hooks irrelevant to PNK's builder role

**Action:** ⚠️ **By design** - PNK intentionally lightweight

---

## What Needs to Be Aligned for Hybrid Inference?

### ✅ Already Aligned (Sufficient for Deployment)

1. **Infrastructure (100%)**
   - Ollama service accessible
   - Native llama-server accessible
   - Both engines can route inference

2. **Skills (100%)**
   - Symlinked directory
   - All 128 skills shared
   - No sync needed

3. **Core Hybrid Files (100%)**
   - HybridInference.ts in both
   - Validation tools in both
   - Configuration tools in both

### ⚠️ Should Consider for Full Alignment

**Inference.ts Integration:**

Currently:
- PNC has full `Inference.ts` (central router)
- PNK has `Inference.ts` but may be outdated

**Check if PNK's Inference.ts needs hybrid integration:**

<function_calls>
<invoke name="mcp_Bash">
<parameter name="command">diff ~/.claude/PAI/Tools/Inference.ts ~/.opencode/PAI/Tools/Inference.ts 2>&1 | head -20