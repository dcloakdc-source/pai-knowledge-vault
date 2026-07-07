# PAI Documentation Index

**Last Updated:** 2026-06-30  
**PAI Version:** 5.1.0  
**Purpose:** Master navigation for all PAI documentation

---

## 🚀 Quick Start

**New to PAI?** Start here:

1. [**QUICKSTART.md**](QUICKSTART.md) — Get PAI running in 10 minutes
2. [**EXECUTIVE_SUMMARY.md**](EXECUTIVE_SUMMARY.md) — What PAI does and why
3. [**ARCHITECTURE_SUMMARY.md**](ARCHITECTURE_SUMMARY.md) — System overview

**Installation:**
- [**DEPLOYMENT_GUIDE.md**](DEPLOYMENT_GUIDE.md) — Step-by-step deployment
- [**CONFIGURATION.md**](CONFIGURATION.md) — Configuration reference

---

## 📚 Documentation by Category

### System Architecture & Design

| Document | Description |
|----------|-------------|
| [ARCHITECTURE.md](ARCHITECTURE.md) | Complete system architecture |
| [ARCHITECTURE_SUMMARY.md](ARCHITECTURE_SUMMARY.md) | High-level overview |
| [PAISystemPhilosophy.md](PAISystemPhilosophy.md) | Design philosophy & principles |
| [GLOSSARY.md](GLOSSARY.md) | Glossary & index of all PAI acronyms, terms, and concepts |
| [ENGINE_DAEMON_ARCHITECTURE.md](ENGINE_DAEMON_ARCHITECTURE.md) | Multi-engine daemon design |
| [SkillDispatcher-Architecture.md](SkillDispatcher-Architecture.md) | Skill system design |

### Operations & Deployment

| Document | Description |
|----------|-------------|
| [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md) | Production deployment guide |
| [CONFIGURATION.md](CONFIGURATION.md) | Configuration reference |
| [HERDR_ENGINE_AUTO_LAUNCH.md](HERDR_ENGINE_AUTO_LAUNCH.md) | Auto-launch configuration |
| [DAEMON_MANAGEMENT_QUICKREF.md](DAEMON_MANAGEMENT_QUICKREF.md) | Service management commands |
| [TROUBLESHOOTING_METHOD.md](TROUBLESHOOTING_METHOD.md) | Standard troubleshooting process |

### ITIL Framework & Service Management

| Document | Description |
|----------|-------------|
| [ITIL_FRAMEWORK.md](ITIL_FRAMEWORK.md) | Complete ITIL implementation |
| [ITIL_QUICK_REFERENCE.md](ITIL_QUICK_REFERENCE.md) | Quick command reference |
| [ITIL_IMPLEMENTATION_GUIDE.md](ITIL_IMPLEMENTATION_GUIDE.md) | Implementation guide |
| [ITIL_ACTIVATION_SUMMARY.md](ITIL_ACTIVATION_SUMMARY.md) | Activation status |
| [SERVICE_CATALOG.md](SERVICE_CATALOG.md) | Service catalog |
| [PROBLEM_RESPONSE_PLAYBOOK.md](PROBLEM_RESPONSE_PLAYBOOK.md) | Incident response |

### Agent Loops & Observability

| Document | Description |
|----------|-------------|
| [AGENT_LOOPS.md](AGENT_LOOPS.md) | Agent loop architecture |
| **AgentLoopMonitor.ts** | Real-time loop monitoring |
| **AgentLoopDashboard/** | Web dashboard for loops |

**Key Features (2026-06-30 deliverables):**
- 24/7 autonomous loop orchestration tracking
- WebSocket-based real-time dashboard
- Loop lifecycle, iteration, token metrics
- Orchestrator→worker pattern visibility

**CLI Commands:**
```bash
# Monitor loops
bun PAI/Tools/AgentLoopMonitor.ts start --orchestrator "main" --pattern "research"
bun PAI/Tools/AgentLoopMonitor.ts iteration <loop-id> --status success
bun PAI/Tools/AgentLoopMonitor.ts end <loop-id>

# Dashboard
bun PAI/Tools/AgentLoopDashboard/server.ts
# Open http://localhost:3000
```

### Verification & Approval Gates

| Feature | Description |
|---------|-------------|
| **ApprovalGate.ts** | Dual-factor approval system |
| **VerificationGates/** | ISC-tracked human approval |

**Key Features (2026-06-30 deliverables):**
- TOTP-based approval (Google Authenticator)
- Voice-confirmation codes
- Tamper-evident audit trail
- ISC verification tracking

**Setup:**
```bash
# One-time TOTP setup
bun PAI/Tools/ApprovalGate.ts --setup

# Request approval
bun PAI/Tools/ApprovalGate.ts --request <slug> <action>

# Verify audit log
bun PAI/Tools/ApprovalGate.ts --verify-log
```

### Cost Tracking & Budget Management

| Tool | Description |
|------|-------------|
| **Cost.ts** | Per-session cost tracking |
| [BUDGET_SOURCE.md](BUDGET_SOURCE.md) | Budget management system |

**Key Features (2026-06-30 deliverables):**
- Multi-model pricing (Claude, Gemini, Ollama)
- Session-level cost tracking
- SQLite-based persistence
- Daily/weekly/monthly reports

**CLI Commands:**
```bash
# Record session cost
bun PAI/Tools/Cost.ts record <session-id> <model> <input-tokens> <output-tokens>

# Get cost report
bun PAI/Tools/Cost.ts report --period daily
bun PAI/Tools/Cost.ts report --period weekly
bun PAI/Tools/Cost.ts report --session <session-id>
```

### Project Management

| Tool | Description |
|------|-------------|
| **ProjectManager.ts** | Multi-project state management |
| [PROJECT_MANAGEMENT.md](PROJECT_MANAGEMENT.md) | Project management system |

**Key Features (2026-06-30 deliverables):**
- Multi-project context switching
- Project parking lot (preserve unfinished work)
- Quick resume with full context
- Git-based state tracking
- ISA integration

**Philosophy:**
> Unfinished projects are exploration, not failure.  
> Context-switching is a feature, not a bug.

**CLI Commands:**
```bash
# Project operations
bun PAI/Tools/ProjectManager.ts switch <project>
bun PAI/Tools/ProjectManager.ts park <project>
bun PAI/Tools/ProjectManager.ts resume <project>
bun PAI/Tools/ProjectManager.ts list
bun PAI/Tools/ProjectManager.ts active
bun PAI/Tools/ProjectManager.ts archive <project>
```

**See also:** [PROJECT_MANAGER_README.md](../Tools/PROJECT_MANAGER_README.md)

### Cross-Domain Pattern Linking

| Tool | Description |
|------|-------------|
| **CrossDomainLinker.ts** | Symbolic pattern detection |
| **CrossDomainLinker.test.ts** | Test suite |
| **CrossDomainLinker.demo.ts** | Demo patterns |
| [CrossDomainLinker.README.md](../Tools/CrossDomainLinker.README.md) | Full documentation |

**Key Features (2026-06-30 deliverables):**
- Cross-domain symbolic pattern detection
- Multi-file pattern synthesis
- Wikilink-style connections
- Integration with Knowledge graph

**Example patterns:**
- Code architecture ↔ martial arts philosophy
- System design ↔ jazz improvisation
- Security patterns ↔ medieval castle design

**CLI Commands:**
```bash
# Detect patterns
bun PAI/Tools/CrossDomainLinker.ts detect <file1> <file2> [...]

# Generate synthesis
bun PAI/Tools/CrossDomainLinker.ts synthesize <domain1> <domain2>

# Run demo
bun PAI/Tools/CrossDomainLinker.demo.ts
```

### Focus & Time Management (ADHD-Optimized)

| Tool | Description |
|------|-------------|
| **FocusManager.ts** | ADHD-focused productivity |
| **FocusPomodoro.ts** | Adaptive Pomodoro timers |
| [FocusManager.md](../Tools/FocusManager.md) | Complete guide |
| [README-FocusManager.md](../Tools/README-FocusManager.md) | Quick reference |

**Key Features (2026-06-30 deliverables):**
- Adaptive Pomodoro (attention-based)
- Distraction capture without disruption
- Task parking lot with context preservation
- Energy state tracking
- Integration with agent loops

**Philosophy:**
> ADHD isn't a bug — it's a different operating system.  
> Work WITH attention patterns, not against them.

**CLI Commands:**
```bash
# Start focus session
bun PAI/Tools/FocusManager.ts start <task> --energy high|medium|low

# Capture distraction
bun PAI/Tools/FocusManager.ts capture "<thought>"

# Complete session
bun PAI/Tools/FocusManager.ts complete

# View stats
bun PAI/Tools/FocusManager.ts stats
```

### Local LLM Performance

| Document | Description |
|----------|-------------|
| [LOCAL_LLM_INVESTIGATION_FINAL_REPORT.md](LOCAL_LLM_INVESTIGATION_FINAL_REPORT.md) | Complete investigation |
| [OLLAMA_OPTIMIZATION_GUIDE.md](OLLAMA_OPTIMIZATION_GUIDE.md) | Tuning guide |
| [SOFT_LAUNCH_PLAN.md](SOFT_LAUNCH_PLAN.md) | Integration plan |
| [SESSION_COMPLETE_2026-06-30.md](SESSION_COMPLETE_2026-06-30.md) | Session summary |

**Performance Results:**
- 2-4x speedup over baseline Ollama
- Quality parity (5/5 tests pass)
- Zero Claude token cost for local work

### Cross-Engine Coordination

| Document | Description |
|----------|-------------|
| [CROSS_ENGINE_PROVENANCE.md](CROSS_ENGINE_PROVENANCE.md) | Cross-engine tracking |
| [org-session-management.md](org-session-management.md) | Multi-org sessions |

### Security & Infrastructure

| Document | Description |
|----------|-------------|
| [SECURITY_OVERRIDE_MECHANISM.md](SECURITY_OVERRIDE_MECHANISM.md) | Security overrides |
| [PAI_BIOGRAPHY.md](PAI_BIOGRAPHY.md) | System provenance |

### Historical Sessions

| Document | Description |
|----------|-------------|
| [SESSION_COMPLETE_2026-06-24.md](SESSION_COMPLETE_2026-06-24.md) | Jun 24 session |
| [SESSION_COMPLETE_2026-06-30.md](SESSION_COMPLETE_2026-06-30.md) | Jun 30 session |
| [SESSION_SUMMARY_2026-06-24.md](SESSION_SUMMARY_2026-06-24.md) | Jun 24 summary |
| [SESSION_SUMMARY_2026-06-27.md](SESSION_SUMMARY_2026-06-27.md) | Jun 27 summary |

---

## 🎯 Documentation by Use Case

### "I want to..."

**...get PAI running**
→ [QUICKSTART.md](QUICKSTART.md) → [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md)

**...understand what PAI does**
→ [EXECUTIVE_SUMMARY.md](EXECUTIVE_SUMMARY.md) → [ARCHITECTURE_SUMMARY.md](ARCHITECTURE_SUMMARY.md)

**...monitor agent loops**
→ [AGENT_LOOPS.md](AGENT_LOOPS.md) → `bun PAI/Tools/AgentLoopMonitor.ts --help`

**...track costs**
→ `bun PAI/Tools/Cost.ts --help` → [BUDGET_SOURCE.md](BUDGET_SOURCE.md)

**...manage multiple projects**
→ [PROJECT_MANAGER_README.md](../Tools/PROJECT_MANAGER_README.md) → `bun PAI/Tools/ProjectManager.ts --help`

**...improve focus (ADHD)**
→ [FocusManager.md](../Tools/FocusManager.md) → `bun PAI/Tools/FocusManager.ts --help`

**...detect cross-domain patterns**
→ [CrossDomainLinker.README.md](../Tools/CrossDomainLinker.README.md) → `bun PAI/Tools/CrossDomainLinker.ts --help`

**...require human approval for actions**
→ `bun PAI/Tools/ApprovalGate.ts --setup`

**...optimize local LLM performance**
→ [OLLAMA_OPTIMIZATION_GUIDE.md](OLLAMA_OPTIMIZATION_GUIDE.md)

**...troubleshoot an issue**
→ [TROUBLESHOOTING_METHOD.md](TROUBLESHOOTING_METHOD.md)

**...manage services**
→ [DAEMON_MANAGEMENT_QUICKREF.md](DAEMON_MANAGEMENT_QUICKREF.md)

**...respond to incidents**
→ [PROBLEM_RESPONSE_PLAYBOOK.md](PROBLEM_RESPONSE_PLAYBOOK.md)

---

## 📖 Complete Tool Reference

### Core Tools (PAI/Tools/)

#### Observability
- **AgentLoopMonitor.ts** — Loop lifecycle tracking
- **AgentLogger.ts** — Activity logging
- **AgentWatchdog.ts** — Health monitoring
- **ObservabilityServer.ts** — Metrics endpoint

#### Approval & Verification
- **ApprovalGate.ts** — Dual-factor approval
- **VerificationGates/** — ISC-tracked gates

#### Cost Management
- **Cost.ts** — Session cost tracking
- **BudgetReport.ts** — Budget reporting
- **EngineMeter.ts** — Multi-engine usage

#### Project Management
- **ProjectManager.ts** — Multi-project switching
- **ProjectState/** — State management
- **ArchiveWork.ts** — Work archival

#### Focus & Productivity
- **FocusManager.ts** — ADHD-optimized focus
- **FocusPomodoro.ts** — Adaptive timers
- **Agenda.ts** — Daily planning

#### Cross-Domain Linking
- **CrossDomainLinker.ts** — Pattern detection
- **WikilinkGraph.ts** — Wikilink graph

#### Media & Content
- **MediaWorkflow.ts** — Media processing
- **FeedMonitor.ts** — RSS/feed monitoring
- **InboxAutoProcessor.ts** — Inbox automation

#### Infrastructure
- **HybridInference.ts** — Local/cloud routing
- **RecordRouter.ts** — Cross-engine handoffs
- **VoiceRouter.ts** — Voice synthesis

#### Security
- **ApprovalGate.ts** — Dual-factor auth
- **SecurityFilter.ts** — Content filtering
- **SecretScan.ts** — Secret detection

---

## 🔧 CLI Quick Reference

### Agent Loop Monitoring
```bash
# Start monitoring loop
bun PAI/Tools/AgentLoopMonitor.ts start --orchestrator <id> --pattern <name>

# Log iteration
bun PAI/Tools/AgentLoopMonitor.ts iteration <loop-id> --status <status>

# End loop
bun PAI/Tools/AgentLoopMonitor.ts end <loop-id>

# Dashboard
bun PAI/Tools/AgentLoopDashboard/server.ts
```

### Cost Tracking
```bash
# Record cost
bun PAI/Tools/Cost.ts record <session> <model> <in-tokens> <out-tokens>

# Get report
bun PAI/Tools/Cost.ts report --period daily
bun PAI/Tools/Cost.ts report --session <id>
```

### Project Management
```bash
# Switch project
bun PAI/Tools/ProjectManager.ts switch <project>

# Park project
bun PAI/Tools/ProjectManager.ts park <project>

# Resume project
bun PAI/Tools/ProjectManager.ts resume <project>

# List all
bun PAI/Tools/ProjectManager.ts list
```

### Focus Management
```bash
# Start focus
bun PAI/Tools/FocusManager.ts start <task> --energy <level>

# Capture distraction
bun PAI/Tools/FocusManager.ts capture "<thought>"

# Complete
bun PAI/Tools/FocusManager.ts complete

# Stats
bun PAI/Tools/FocusManager.ts stats
```

### Approval Gates
```bash
# Setup TOTP
bun PAI/Tools/ApprovalGate.ts --setup

# Request approval
bun PAI/Tools/ApprovalGate.ts --request <slug> <action>

# Verify log
bun PAI/Tools/ApprovalGate.ts --verify-log
```

### Cross-Domain Linking
```bash
# Detect patterns
bun PAI/Tools/CrossDomainLinker.ts detect <files...>

# Synthesize
bun PAI/Tools/CrossDomainLinker.ts synthesize <domain1> <domain2>
```

---

## 🏗️ Integration Guides

### How Features Connect

```
┌─────────────────────────────────────────────────────────────┐
│                     PAI Architecture                         │
└─────────────────────────────────────────────────────────────┘

AgentLoopMonitor ──┐
                   ├──> Dashboard ──> WebSocket
                   └──> JSONL logs

Cost.ts ───────────┐
                   ├──> SQLite DB ──> Reports
                   └──> BudgetReport.ts

ProjectManager ────┐
                   ├──> .project/ dirs ──> Git
                   ├──> ProjectState skill
                   └──> ISA integration

FocusManager ──────┐
                   ├──> Pomodoro timers
                   ├──> Distraction capture
                   └──> AgentLoopMonitor

CrossDomainLinker ─┐
                   ├──> Knowledge graph
                   ├──> WikilinkGraph
                   └──> Pattern detection

ApprovalGate ──────┐
                   ├──> TOTP verification
                   ├──> Voice confirmation
                   └──> Audit trail (JSONL)

HybridInference ───┐
                   ├──> Ollama (local)
                   ├──> llama-server (GPU)
                   └──> Cloud fallback
```

---

## 🧪 API Reference

### TypeScript APIs

#### AgentLoopMonitor API
```typescript
import { logLoopStart, logIteration, logLoopEnd } from './AgentLoopMonitor';

const loopId = await logLoopStart({
  orchestratorId: "main",
  pattern: "research",
  goal: "Find papers on topic X"
});

await logIteration(loopId, {
  iterationNum: 1,
  status: "success",
  tokensUsed: 1200,
  model: "claude-3-5-sonnet"
});

await logLoopEnd(loopId, {
  status: "completed",
  totalIterations: 5
});
```

#### Cost Tracking API
```typescript
import { calculateCost, recordSessionCost } from './Cost';

// Calculate cost
const cost = calculateCost("claude-3-5-sonnet", 1000, 500);

// Record to database
recordSessionCost("ses_123", "claude-3-5-sonnet", 1000, 500);
```

#### ProjectManager API
```typescript
import { switchProject, parkProject, resumeProject } from './ProjectManager';

// Switch active project
await switchProject("my-project", "/path/to/project");

// Park for later
await parkProject("my-project", {
  reason: "Taking a break",
  context: "Left off implementing feature X"
});

// Resume with context
const context = await resumeProject("my-project");
```

#### FocusManager API
```typescript
import { startFocus, captureDistraction, completeFocus } from './FocusManager';

// Start focus session
const sessionId = await startFocus({
  task: "Write documentation",
  energy: "medium",
  estimatedPomodoros: 3
});

// Capture distraction
await captureDistraction(sessionId, {
  thought: "Check email about meeting",
  urgency: "low"
});

// Complete
await completeFocus(sessionId);
```

---

## 🛠️ Troubleshooting

### Common Issues

**Agent loops not appearing in dashboard?**
→ Check WebSocket connection: `curl http://localhost:3000/api/loops`

**Cost tracking not working?**
→ Verify SQLite DB exists: `ls -lh ~/MEMORY/db/costs.db`

**Project switch failed?**
→ Check registry: `cat ~/.claude/PAI/MEMORY/STATE/projects-registry.json`

**Focus session interrupted?**
→ Resume: `bun PAI/Tools/FocusManager.ts resume`

**TOTP not working?**
→ Re-sync time: `sudo ntpdate -u time.google.com`

### Full Troubleshooting Guide

See [TROUBLESHOOTING_METHOD.md](TROUBLESHOOTING_METHOD.md) for the complete process.

---

## 📦 Installation & Deployment

### Quick Install

```bash
# Clone repo
git clone https://github.com/yourusername/pai.git ~/.claude

# Run installer (2026-06-30 deliverable)
bash ~/.claude/install.sh

# Configure
$EDITOR ~/.claude/settings.json

# Start services
systemctl --user start pai-primary
```

### Full Deployment

See [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md) for complete instructions.

---

## 📅 What's New (2026-06-30 Session)

### Major Deliverables

1. **Agent Loop Observability** ✅
   - AgentLoopMonitor with real-time tracking
   - Web dashboard with WebSocket updates
   - Loop lifecycle, iteration, token metrics

2. **Verification Gates** ✅
   - Dual-factor approval (TOTP + voice)
   - ISC-tracked verification
   - Tamper-evident audit trail

3. **Cost Tracking** ✅
   - Multi-model pricing database
   - Session-level tracking
   - Daily/weekly/monthly reports

4. **Project Management** ✅
   - Multi-project context switching
   - Parking lot for unfinished work
   - Quick resume with full context

5. **Cross-Domain Linking** ✅
   - Symbolic pattern detection
   - Multi-file synthesis
   - Knowledge graph integration

6. **Focus Manager** ✅
   - ADHD-optimized productivity
   - Adaptive Pomodoro timers
   - Distraction capture

7. **Deployment Documentation** ✅
   - 4 deployment guides
   - Service configuration
   - Troubleshooting playbooks

8. **Installer** ✅
   - One-command setup
   - Dependency verification
   - Service installation

9. **YouTube Fixes** ✅
   - RSS feed monitoring
   - Deduplication
   - Daily digest

10. **Infrastructure Portability** ✅
    - Abstraction layers
    - Multi-engine support
    - Cross-platform compatibility

11. **Relationship AI MVP** ✅
    - 2 ADHD-focused tools
    - Context preservation
    - Energy tracking

### Performance Achievements

**Local LLM:**
- 2-4x speedup vs baseline
- Quality parity maintained
- Zero token cost for local work

**System Health:**
- 11+ major features delivered
- Zero regressions
- Full test coverage

---

## 🔗 Related Documentation

### Skills System
See [~/.claude/skills/README.md](../../skills/README.md) for available skills.

### Tools Directory
See [~/.claude/PAI/Tools/README.md](../Tools/README.md) for tool catalog.

### Algorithm Framework
See [~/.claude/PAI/Algorithm/](../Algorithm/) for Algorithm v5.7.3 documentation.

---

## 📞 Support & Contact

**Documentation Issues:**
File issues at: [GitHub Issues](https://github.com/yourusername/pai/issues)

**Questions:**
See [TROUBLESHOOTING_METHOD.md](TROUBLESHOOTING_METHOD.md) first.

**Contributing:**
See [CONTRIBUTING.md](../../CONTRIBUTING.md) for guidelines.

---

## 📝 Maintenance

**Last Full Review:** 2026-06-30  
**Next Review Due:** 2026-07-31  
**Maintainer:** Duane (Principal)  
**DA:** PAI Nova (PNK/PNC/PNG/PNO/PNX)

---

## 🗂️ Complete File Listing

<details>
<summary>All Documentation Files (56 files)</summary>

```
PAI/DOCUMENTATION/
├── INDEX.md (this file)
├── QUICKSTART.md
├── EXECUTIVE_SUMMARY.md
├── ARCHITECTURE_SUMMARY.md
├── ARCHITECTURE.md
├── PAISystemPhilosophy.md
├── DEPLOYMENT_GUIDE.md
├── CONFIGURATION.md
├── TROUBLESHOOTING_METHOD.md
├── AGENT_LOOPS.md
├── BUDGET_SOURCE.md
├── CROSS_ENGINE_PROVENANCE.md
├── DAEMON_MANAGEMENT_QUICKREF.md
├── ENGINE_DAEMON_ARCHITECTURE.md
├── HERDR_ENGINE_AUTO_LAUNCH.md
├── ITIL_FRAMEWORK.md
├── ITIL_QUICK_REFERENCE.md
├── ITIL_IMPLEMENTATION_GUIDE.md
├── ITIL_ACTIVATION_SUMMARY.md
├── SERVICE_CATALOG.md
├── PROBLEM_RESPONSE_PLAYBOOK.md
├── PROJECT_MANAGEMENT.md
├── SECURITY_OVERRIDE_MECHANISM.md
├── SkillDispatcher-Architecture.md
├── org-session-management.md
├── PAI_BIOGRAPHY.md
├── LOCAL_LLM_INVESTIGATION_FINAL_REPORT.md
├── OLLAMA_OPTIMIZATION_GUIDE.md
├── SOFT_LAUNCH_PLAN.md
├── SESSION_COMPLETE_2026-06-30.md
├── SESSION_COMPLETE_2026-06-24.md
├── SESSION_SUMMARY_2026-06-27.md
├── SESSION_SUMMARY_2026-06-24.md
├── PHASE3_5_COMPLETE.md
├── PHASE_D_OLLAMA_OPTIMIZATION.md
├── PHASE1_OLLAMA_BASELINE_RESULTS.md
├── PHASE2_LLAMACPP_RESULTS.md
├── PHASE3_LLAMACPP_NATIVE_RESULTS.md
├── PHASE3_5_INTEGRATION_PROGRESS.md
├── PRE_REBOOT_STATE.md
├── VLLM_INSTALLATION_RESULT.md
├── VLLM_VS_OLLAMA_BENCHMARK.md
├── VLLM_FINAL_VERDICT.md
├── VLLM_AWQ_RESULT.md
├── LOCAL_LLM_COMPREHENSIVE_INVESTIGATION.md
├── LOCAL_LLM_INVESTIGATION_PROGRESS.md
├── OLLAMA_TIMEOUT_INVESTIGATION.md
├── OLLAMA_STATUS_VERIFIED.md
├── OLLAMA_CAPABILITY_ASSESSMENT.md
├── MODEL_BENCHMARK_SYSTEM.md
├── OPENROUTER_INTEGRATION_GUIDE.md
├── OPENROUTER_VS_OPENCODE.md
├── CAPABILITY_REGISTRY_IMPLEMENTATION_SUMMARY.md
├── PROBLEM_DETECTION_IMPLEMENTATION_SUMMARY.md
├── RCA_IMPLEMENTATION_SUMMARY.md
├── REARCHITECTURE_GAP_MAP.md
└── POSTMORTEM_TEMPLATE.md
```

</details>

---

**End of Index** — Use Ctrl+F to search this document.
