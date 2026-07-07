# PAI Documentation Navigation Map

**Visual guide to finding documentation**  
**Last Updated:** 2026-06-30

---

## 🗺️ Visual Navigation

```
┌─────────────────────────────────────────────────────────────────┐
│                    PAI DOCUMENTATION HUB                         │
│                                                                  │
│  Start Here: INDEX.md                                           │
│  Quick Ref:  QUICK_REFERENCE_CARD.md                           │
│  This Map:   NAVIGATION_MAP.md                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
   🚀 NEW USER         📖 OPERATIONS          🔧 DEVELOPER
        │                     │                     │
        │                     │                     │
        ▼                     ▼                     ▼
┌──────────────┐     ┌──────────────┐      ┌──────────────┐
│ QUICKSTART   │     │ DEPLOYMENT   │      │ ARCHITECTURE │
│ EXECUTIVE    │     │ CONFIG       │      │ API DOCS     │
│ SUMMARY      │     │ ITIL GUIDES  │      │ TOOL REFS    │
└──────────────┘     └──────────────┘      └──────────────┘
```

---

## 🎯 By Role

### New User
**Goal:** Get PAI running and understand what it does

**Path:**
1. [INDEX.md](INDEX.md) → "Quick Start" section
2. [QUICKSTART.md](QUICKSTART.md) → Step-by-step setup
3. [EXECUTIVE_SUMMARY.md](EXECUTIVE_SUMMARY.md) → What PAI does
4. [QUICK_REFERENCE_CARD.md](QUICK_REFERENCE_CARD.md) → Common commands

**Time to productive:** 30 minutes

---

### Operator
**Goal:** Deploy, monitor, and maintain PAI

**Path:**
1. [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md) → Production deployment
2. [CONFIGURATION.md](CONFIGURATION.md) → Configuration reference
3. [DAEMON_MANAGEMENT_QUICKREF.md](DAEMON_MANAGEMENT_QUICKREF.md) → Service commands
4. [TROUBLESHOOTING_METHOD.md](TROUBLESHOOTING_METHOD.md) → When things break
5. [ITIL_QUICK_REFERENCE.md](ITIL_QUICK_REFERENCE.md) → ITIL workflows

**Essential tools:**
- `systemctl --user status pai-primary`
- `bun PAI/Tools/HealthCheck.ts`
- `journalctl --user -u pai-primary -f`

---

### Developer
**Goal:** Extend PAI or integrate with it

**Path:**
1. [ARCHITECTURE.md](ARCHITECTURE.md) → System design
2. [SkillDispatcher-Architecture.md](SkillDispatcher-Architecture.md) → Skills system
3. [INDEX.md](INDEX.md) → "API Reference" section
4. [ENGINE_DAEMON_ARCHITECTURE.md](ENGINE_DAEMON_ARCHITECTURE.md) → Multi-engine design

**Essential tools:**
- TypeScript APIs in PAI/Tools/
- Skills in ~/.claude/skills/
- MCP servers in PAI/Tools/

---

### Power User
**Goal:** Use all features effectively

**Path:**
1. [INDEX.md](INDEX.md) → "Documentation by Use Case"
2. [QUICK_REFERENCE_CARD.md](QUICK_REFERENCE_CARD.md) → Print for desk
3. Tool-specific READMEs:
   - [PROJECT_MANAGER_README.md](../Tools/PROJECT_MANAGER_README.md)
   - [FocusManager.md](../Tools/FocusManager.md)
   - [CrossDomainLinker.README.md](../Tools/CrossDomainLinker.README.md)

**Workflow:**
- Morning: Focus session + project switch
- During: Agent loop monitoring
- Evening: Cost report + stats

---

## 📊 By Feature

```
┌────────────────────────────────────────────────────────────┐
│                    FEATURE MAP                              │
└────────────────────────────────────────────────────────────┘

Agent Loops ──┐
              ├─→ AGENT_LOOPS.md
              ├─→ AgentLoopMonitor.ts --help
              └─→ Dashboard: http://localhost:3000

Cost Tracking ┐
              ├─→ Cost.ts --help
              ├─→ BUDGET_SOURCE.md
              └─→ QUICK_REFERENCE_CARD.md → "Cost Tracking"

Projects ─────┐
              ├─→ PROJECT_MANAGEMENT.md
              ├─→ PROJECT_MANAGER_README.md
              └─→ ProjectManager.ts --help

Focus (ADHD) ─┐
              ├─→ FocusManager.md
              ├─→ README-FocusManager.md
              └─→ FocusManager.ts --help

Approval ─────┐
              ├─→ ApprovalGate.ts --setup
              └─→ QUICK_REFERENCE_CARD.md → "Approval Gates"

Patterns ─────┐
              ├─→ CrossDomainLinker.README.md
              └─→ CrossDomainLinker.demo.ts

Local LLM ────┐
              ├─→ LOCAL_LLM_INVESTIGATION_FINAL_REPORT.md
              ├─→ OLLAMA_OPTIMIZATION_GUIDE.md
              └─→ SOFT_LAUNCH_PLAN.md

ITIL ─────────┐
              ├─→ ITIL_FRAMEWORK.md
              ├─→ ITIL_QUICK_REFERENCE.md
              └─→ PROBLEM_RESPONSE_PLAYBOOK.md
```

---

## 🔍 By Task

### "I want to..."

#### ...get started
→ [QUICKSTART.md](QUICKSTART.md)

#### ...understand the system
→ [ARCHITECTURE_SUMMARY.md](ARCHITECTURE_SUMMARY.md)

#### ...deploy to production
→ [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md)

#### ...configure PAI
→ [CONFIGURATION.md](CONFIGURATION.md)

#### ...monitor agent loops
→ [AGENT_LOOPS.md](AGENT_LOOPS.md)

#### ...track costs
→ `bun PAI/Tools/Cost.ts --help`

#### ...manage projects
→ [PROJECT_MANAGER_README.md](../Tools/PROJECT_MANAGER_README.md)

#### ...improve focus (ADHD)
→ [FocusManager.md](../Tools/FocusManager.md)

#### ...detect patterns
→ [CrossDomainLinker.README.md](../Tools/CrossDomainLinker.README.md)

#### ...require approval
→ `bun PAI/Tools/ApprovalGate.ts --setup`

#### ...optimize performance
→ [OLLAMA_OPTIMIZATION_GUIDE.md](OLLAMA_OPTIMIZATION_GUIDE.md)

#### ...troubleshoot
→ [TROUBLESHOOTING_METHOD.md](TROUBLESHOOTING_METHOD.md)

#### ...respond to incident
→ [PROBLEM_RESPONSE_PLAYBOOK.md](PROBLEM_RESPONSE_PLAYBOOK.md)

---

## 📁 Directory Structure

```
~/.claude/PAI/DOCUMENTATION/
│
├── INDEX.md ⭐                    # Start here
├── QUICK_REFERENCE_CARD.md ⭐     # Print this
├── NAVIGATION_MAP.md ⭐            # This file
│
├── Quick Start/
│   ├── QUICKSTART.md
│   ├── EXECUTIVE_SUMMARY.md
│   └── ARCHITECTURE_SUMMARY.md
│
├── Operations/
│   ├── DEPLOYMENT_GUIDE.md
│   ├── CONFIGURATION.md
│   ├── DAEMON_MANAGEMENT_QUICKREF.md
│   └── TROUBLESHOOTING_METHOD.md
│
├── ITIL/
│   ├── ITIL_FRAMEWORK.md
│   ├── ITIL_QUICK_REFERENCE.md
│   ├── ITIL_IMPLEMENTATION_GUIDE.md
│   ├── ITIL_ACTIVATION_SUMMARY.md
│   ├── SERVICE_CATALOG.md
│   └── PROBLEM_RESPONSE_PLAYBOOK.md
│
├── Architecture/
│   ├── ARCHITECTURE.md
│   ├── ENGINE_DAEMON_ARCHITECTURE.md
│   ├── SkillDispatcher-Architecture.md
│   └── PAISystemPhilosophy.md
│
├── Features/
│   ├── AGENT_LOOPS.md
│   ├── BUDGET_SOURCE.md
│   ├── PROJECT_MANAGEMENT.md
│   ├── CROSS_ENGINE_PROVENANCE.md
│   └── SECURITY_OVERRIDE_MECHANISM.md
│
├── Performance/
│   ├── LOCAL_LLM_INVESTIGATION_FINAL_REPORT.md
│   ├── OLLAMA_OPTIMIZATION_GUIDE.md
│   └── SOFT_LAUNCH_PLAN.md
│
└── Sessions/
    ├── SESSION_COMPLETE_2026-06-30.md
    ├── SESSION_COMPLETE_2026-06-24.md
    ├── SESSION_SUMMARY_2026-06-27.md
    └── SESSION_SUMMARY_2026-06-24.md
```

---

## 🎓 Learning Paths

### Path 1: Quick Start (1 hour)
1. QUICKSTART.md (15 min)
2. Install PAI (20 min)
3. QUICK_REFERENCE_CARD.md (10 min)
4. Try first command (15 min)

**Outcome:** PAI running, basic commands known

---

### Path 2: Power User (4 hours)
1. EXECUTIVE_SUMMARY.md (20 min)
2. ARCHITECTURE_SUMMARY.md (30 min)
3. Feature docs (2 hours):
   - Agent loops
   - Project management
   - Focus management
   - Cost tracking
4. Hands-on practice (1 hour)

**Outcome:** Proficient with all major features

---

### Path 3: Operator (8 hours)
1. Deployment (2 hours):
   - DEPLOYMENT_GUIDE.md
   - CONFIGURATION.md
   - Install to production
2. ITIL Framework (2 hours):
   - ITIL_FRAMEWORK.md
   - ITIL_IMPLEMENTATION_GUIDE.md
3. Operations (2 hours):
   - DAEMON_MANAGEMENT_QUICKREF.md
   - TROUBLESHOOTING_METHOD.md
   - PROBLEM_RESPONSE_PLAYBOOK.md
4. Practice scenarios (2 hours)

**Outcome:** Can deploy and maintain PAI in production

---

### Path 4: Developer (16 hours)
1. Architecture (4 hours):
   - ARCHITECTURE.md
   - ENGINE_DAEMON_ARCHITECTURE.md
   - SkillDispatcher-Architecture.md
2. API exploration (4 hours):
   - TypeScript APIs
   - MCP servers
   - Tool CLIs
3. Code reading (4 hours):
   - PAI/Tools/ codebase
   - Skills system
   - Hook system
4. Build extension (4 hours):
   - Create custom skill
   - Write MCP server
   - Add new tool

**Outcome:** Can extend PAI with custom features

---

## 🔄 Update Cycle

**Documentation is living:**

```
User feedback ──┐
                ├──→ Update docs
New features ───┤
                ├──→ Regenerate INDEX.md
Improvements ───┘
                └──→ Update QUICK_REFERENCE_CARD.md
```

**Review schedule:**
- Quick Start docs: Monthly
- Feature docs: After each major release
- API docs: On API changes
- Full index: Monthly

---

## 📞 Finding Help

```
┌─────────────────────────────────────────┐
│         HELP DECISION TREE              │
└─────────────────────────────────────────┘

Need help?
    │
    ├─→ Quick command? → QUICK_REFERENCE_CARD.md
    │
    ├─→ How to do X? → INDEX.md → "I want to..."
    │
    ├─→ Understanding concept? → Feature docs
    │
    ├─→ Something broken? → TROUBLESHOOTING_METHOD.md
    │
    ├─→ Incident? → PROBLEM_RESPONSE_PLAYBOOK.md
    │
    └─→ Still stuck? → GitHub Issues
```

---

## 🎯 Essential Trio

**Print and keep these three:**

1. **QUICK_REFERENCE_CARD.md** — Daily commands
2. **INDEX.md** — Master navigation
3. **This file (NAVIGATION_MAP.md)** — Find anything

---

## 📊 Documentation Stats

**As of 2026-06-30:**

- Total docs: 56 files
- Total sections: 99+
- Total references: 114+ links
- Quick Start path: 30 minutes
- Power User path: 4 hours
- Operator path: 8 hours
- Developer path: 16 hours

---

## 🗂️ Cross-References

**This document connects to:**
- [INDEX.md](INDEX.md) — Master index
- [QUICK_REFERENCE_CARD.md](QUICK_REFERENCE_CARD.md) — Command reference
- All feature documentation
- All tool READMEs

**See INDEX.md for complete file listing.**

---

**Navigation Version:** 1.0.0  
**Last Updated:** 2026-06-30  
**Maintainer:** Duane (Principal)  
**DA:** PAI Nova (PNK)
