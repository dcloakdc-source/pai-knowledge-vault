# PNC Capabilities Reference — For PNO/PNG/PNX/PNK

**Purpose:** Cross-engine capability awareness for optimal routing  
**Updated:** 2026-07-05  
**Version:** PAI 5.1.0

---

## 🎯 Quick Routing Guide

| Task Type | Route To | Why |
|-----------|----------|-----|
| Classification/labeling | PNO (ollama/llama3.2:3b) | Free, instant, sufficient quality |
| JSON extraction | PNO (ollama/qwen2.5:7b) | Free, structured output |
| Summarization | PNO (ollama/gemma2:9b) | Free, good quality |
| Simple reasoning | PNC (claude/sonnet) | Best quality, fast |
| Complex code (multi-file) | PNK (opencode) | Multi-file architecture |
| Web research | PNG (antigravity) | Fast, free (Gemini) |
| Verification/testing | PNX (codex) | Sandboxed execution |
| Algorithm mode | PNC only | Full PAI orchestration |

---

## 📚 PNC Skills (129 available)

### Thinking & Analysis
- **Think** — Unified debate/council router
- **BeCreative** — Extended thinking mode
- **FirstPrinciples** — Assumption decomposition
- **SystemsThinking** — Causal loops, leverage points
- **RootCauseAnalysis** — 5 Whys, Ishikawa, postmortem
- **ApertureOscillation** — 3-pass scope shifting
- **IterativeDepth** — Multi-angle ISC extraction
- **DesignThinking** — 5-phase design process
- **Science** — Hypothesis-test-analyze cycles

### Research & Content
- **YouTube** — Video search, transcript extraction
- **ArXiv** — Paper search with AlphaXiv summaries
- **Research** — Multi-source research orchestration
- **NotebookLM** — Google NotebookLM integration
- **ResearchPipeline** — YouTube + NotebookLM combined
- **ExtractWisdom** — Content-adaptive wisdom extraction
- **Fabric** — 240+ specialized analysis patterns
- **Parser** — URL/file content extraction to JSON

### Development
- **Engineer** — Elite principal engineer agent
- **Architect** — System design specialist
- **Designer** — UX/UI design expert
- **Simplify** — Post-change code quality review
- **CodeReviewer-Agency** — Thorough code review
- **CreateCLI** — TypeScript CLI generator
- **MCPBuilder** — MCP server design specialist
- **BitterPillEngineering** — Instruction set audit

### Security
- **SecurityTriage** — Adversarial finding verification
- **PAISecurityAudit** — STRIDE threat modeling for PAI
- **InfoSecRiskAssessment** — Vendor/SaaS risk assessment
- **WebAssessment** — Web security testing
- **PromptInjection** — Jailbreak testing
- **Recon** — Security reconnaissance

### Project & Task Management
- **ISA** — Ideal State Artifact creation/management
- **PIGSD** — pi-gsd project delivery framework
- **ProjectState** — Project-scoped state management
- **PhaseRoadmap** — Multi-phase project roadmapping
- **PRWorkflow** — Automated pull request creation
- **DeployOrchestrator** — Deploy→smoke test→verify→rollback
- **Delegation** — Multi-agent parallel execution
- **Loop** — Iterative improvement cycles
- **Optimize** — Autonomous optimization loop

### Infrastructure & Deployment
- **Cloudflare** (ecosystem) — Workers, Pages, R2, D1, KV, Email
- **agents-sdk** — Cloudflare Agents SDK
- **durable-objects** — Durable Objects patterns
- **workers-best-practices** — Production hardening
- **wrangler** — Wrangler CLI reference
- **cloudflare-email-service** — Email Sending + Routing
- **sandbox-sdk** — Sandboxed code execution

### Content Creation
- **WriteStory** — Fiction writing with Storr's method
- **Art** — Visual content creation (Flux, GPT-Image)
- **Webdesign** — Claude Design + frontend-design
- **Remotion** — Programmatic video with React
- **AudioEditor** — Whisper→Claude→ffmpeg pipeline
- **Excalidraw** — Diagram generation
- **Spatial3D** — 3D model from floor plans

### Testing & Quality
- **QATester** — Browser-based UI validation
- **UIReviewer** — User story validation with Playwright
- **RealityChecker** — Skeptical quality gate
- **Evals** — Agent evaluation framework
- **UXAudit** — Pre-delivery UX checklist
- **DesignCritique** — Nielsen heuristics review

### Knowledge & Memory
- **Knowledge** — Typed graph (People/Companies/Ideas)
- **Telos** — Life goals, projects, dependencies
- **Interview** — Phased conversational context intake
- **Migrate** — Content migration into PAI
- **ContextSearch** — Search PRDs, git, sessions
- **WorkCommand** — WORK session recall

### PAI System
- **OllamaSkill** — Local inference routing (THIS IS KEY)
- **AntigravitySkill** — Web research dispatch
- **PAIUpgrade** — Extract system improvements
- **HealthCheck** — Infrastructure health probe
- **SettingsLab** — Self-improving settings system
- **BitterPillEngineering** — Over-prompting audit

### Specialized
- **Ideate** — Evolutionary ideation engine (9 phases)
- **RedTeam** — 32-agent adversarial analysis
- **MultiEngineSynthesis** — Cross-engine consolidation
- **WorldThreatModel** — 11-horizon future stress test
- **TRVRM** — Vulnerability lifecycle management
- **KeyRotation** — Incident-response key rotation
- **ParentalControl** — PAI-powered child PC management

---

## 🤖 PNC Agents (33 available)

### Development
- **Engineer** — Elite principal engineer (Fortune 10 experience)
- **Architect** — System design specialist (distributed systems PhD-level)
- **Designer** — UX/UI design expert (design school pedigree)
- **BackendDev** — Backend specialist
- **explore** — Fast codebase exploration

### Quality & Testing
- **QATester** — Browser-based functional validation (Gate 4)
- **RealityChecker** — Skeptical quality gate (default: NEEDS WORK)
- **CodeReviewer-Agency** — Thorough code review (blocker/suggestion/nit)
- **UIReviewer** — Playwright user story validation

### Research
- **ClaudeResearcher** — Claude WebSearch, multi-query decomposition
- **GeminiResearcher** — Google Gemini, 3-10 query variations
- **CodexResearcher** — Remy, multi-model consultation (O3/GPT-5/GPT-4)
- **DeepResearcher** — Comprehensive investigation
- **PerplexityResearcher** — Ava, investigative analyst with Perplexity
- **GrokResearcher** — Johannes, contrarian fact-based via xAI Grok

### Specialized
- **Algorithm** — Full PAI Algorithm runner (v5.7.11)
- **AgentsOrchestrator** — Multi-agent pipeline manager
- **ProductManager** — Read-only requirements analyst
- **MCPBuilder** — MCP server design specialist
- **Artist** — Visual content creator (Media skill workflows)
- **Pentester** — Web security assessment (WebAssessment workflows)

### Operations
- **IncidentResponseCommander** — SEV1-SEV4 structured response
- **InfraOps** — Docker, systemd, backup verification
- **DBOps** — Qdrant, ICM memory, schema verification

### Intelligence
- **Intern** — 176 IQ generalist, 5 PhDs, high-agency problem solver

### Browser
- **BrowserAgent** — Parallel headless browser automation (Playwright)

---

## 🔄 Routing Decision Tree

```
Task received
    │
    ├─ Classification/scoring? → PNO (ollama/llama3.2:3b, free)
    ├─ JSON extraction? → PNO (ollama/qwen2.5:7b, free)
    ├─ Summarization? → PNO (ollama/gemma2:9b, free)
    │
    ├─ Web research? → PNG (antigravity, Gemini, free)
    ├─ YouTube search? → Skill('YouTube')
    ├─ Academic papers? → Skill('ArXiv')
    │
    ├─ Simple code edit? → PNC (claude/sonnet)
    ├─ Multi-file refactor? → PNK (opencode)
    ├─ Verification? → PNX (codex sandbox)
    │
    ├─ Security assessment? → Skill('SecurityTriage') or Agent('Pentester')
    ├─ Code review? → Agent('CodeReviewer-Agency') or Skill('Simplify')
    │
    ├─ Skill exists for task? → Skill('[name]')
    ├─ Agent specializes? → Agent('[type]')
    │
    └─ Complex/multi-step? → PNC Algorithm mode
```

---

## 💰 Cost Matrix

| Engine | Model | Cost | Speed | Quality | Use For |
|--------|-------|------|-------|---------|---------|
| PNO | llama3.2:3b | $0 | Fast | Good | Classification |
| PNO | qwen2.5:7b | $0 | Fast | Good | JSON |
| PNO | gemma2:9b | $0 | Fast | Good | Summarization |
| PNG | Gemini | $0 | Fast | Good | Web research |
| PNC | Sonnet | $$$ | Medium | Excellent | Code, reasoning |
| PNK | OpenCode | $$$ | Medium | Excellent | Multi-file |
| PNX | Codex | $$ | Slow | Good | Verification |

---

## 🎯 When PNO Should Escalate to PNC

**Stay local (PNO) when:**
- Task is classification/scoring/labeling
- JSON extraction with clear schema
- Summarization of content
- Simple categorization
- Pattern matching

**Escalate to PNC when:**
- Complex reasoning required
- Code generation needed
- Multi-step workflow
- Skill/agent dispatch needed
- Algorithm mode appropriate
- ICM memory operations
- Cross-engine coordination

**Escalate to PNG when:**
- Web search required
- Real-time information needed
- Multi-query research
- YouTube/web content lookup

**Escalate to PNK when:**
- Multi-file architecture work
- Complex refactoring
- Provider-agnostic coding
- TUI-driven planning needed

**Escalate to PNX when:**
- Sandboxed verification
- Adversarial testing (Cato)
- Isolated execution required

---

## 📋 Capability Check Commands

**List all skills:**
```bash
ls ~/.claude/skills/
```

**List all agents:**
```bash
ls ~/.claude/agents/*.md
```

**Check skill description:**
```bash
grep "^description:" ~/.claude/skills/[name]/SKILL.md
```

**Check routing policy:**
```bash
cat ~/.claude/PAI/Tools/Inference.ts | grep -A 10 "task-profile"
```

---

## 🔧 Integration Points

**For PNO to query PNC capabilities:**
```bash
# Check if skill exists
[ -d ~/.claude/skills/[name] ] && echo "exists"

# Check if agent exists
[ -f ~/.claude/agents/[name].md ] && echo "exists"

# Get routing recommendation
bun ~/.claude/PAI/Tools/Inference.ts --task [profile] "[input]"
```

**For ICM capability lookup:**
```bash
# Store capability metadata
icm store --topic capabilities --key "skill:[name]" --value "{...}"

# Query capability
icm recall --topic capabilities --query "skill:[name]"
```

---

## 🎓 Learning Examples

**Example 1: Classification Task**

Task: "Classify this email as spam or ham"

PNO thinks:
- Classification task
- Check routing → ollama/llama3.2:3b
- Local, free, fast, sufficient quality
- **Decision:** Stay local, use Skill('OllamaSkill', '--task fast_classification')

**Example 2: Multi-File Refactor**

Task: "Refactor authentication across 15 files"

PNO thinks:
- Multi-file code task
- Check routing → complex code
- PNC (single-file) vs PNK (multi-file)
- 15 files = multi-file architecture
- **Decision:** Escalate to PNK via `opencode run "..."`

**Example 3: Web Research**

Task: "Find latest Claude Code features"

PNO thinks:
- Web search needed
- Check routing → research
- PNG (Antigravity) vs Skill('YouTube') vs Skill('ArXiv')
- General web research = PNG
- **Decision:** Escalate to PNG via `agy --dangerously-skip-permissions -p "..."`

---

## 📊 Capability Coverage

| Domain | Skills | Agents | Coverage |
|--------|--------|--------|----------|
| Development | 15+ | 5 | Excellent |
| Security | 8+ | 2 | Good |
| Research | 10+ | 5 | Excellent |
| Thinking | 9+ | 1 | Good |
| Infrastructure | 12+ | 2 | Good |
| Content | 8+ | 2 | Good |
| Testing | 6+ | 3 | Good |
| Project Mgmt | 8+ | 2 | Good |

**Total:** 129 skills + 33 agents = 162 capabilities

---

## 🚀 Quick Reference

**Most commonly needed:**
- Classification → `Skill('OllamaSkill', '--task fast_classification')`
- JSON → `Skill('OllamaSkill', '--task structured_json')`
- Web research → `agy --dangerously-skip-permissions -p "..."`
- Code review → `Agent('CodeReviewer-Agency')` or `Skill('Simplify')`
- Multi-file code → `opencode run "..."`
- Complex task → PNC Algorithm mode

**Check before routing:**
1. Is there a skill for this? (129 options)
2. Is there an agent for this? (33 options)
3. Can PNO handle locally? (classification/JSON/summary)
4. Is web research needed? (PNG)
5. Is it multi-file? (PNK)
6. Else → PNC

---

**This file should be read by PNO/PNG/PNX/PNK at startup for cross-engine capability awareness.**

**Location:** `~/.claude/PAI/PNC_CAPABILITIES_FOR_PNO.md`
