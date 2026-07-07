# PAI Architecture Overview

PAI (Personal AI Infrastructure) is a Life Operating System that magnifies human capabilities through AI. This document explains how the pieces fit together.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        You (Principal)                       │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                   Claude Code Interface                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Terminal    │  │  VS Code     │  │  CLI         │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                      PAI Nova (DA)                           │
│               Digital Assistant Identity                     │
└─┬───────────────────────────────────────────────────────────┘
  │
  ├─► Algorithm (7-phase execution engine)
  ├─► Skills (122 specialized capabilities)
  ├─► Hooks (111 lifecycle interceptors)
  ├─► Memory (5-layer persistent storage)
  ├─► Tools (464 utilities and services)
  └─► Agents (25 specialist types)
```

## Core Components

### 1. The Algorithm

The Algorithm is PAI's execution engine — a 7-phase loop that takes any request from current state to ideal state.

**Phases:**

```
OBSERVE  → Understand the situation and build ISC (Ideal State Criteria)
THINK    → Analyze the problem space
PLAN     → Create an execution strategy
BUILD    → Construct the solution
EXECUTE  → Run the plan
VERIFY   → Confirm success against ISC
LEARN    → Capture insights for future use
```

**Location:** `PAI/Algorithm/`

**Current Version:** Pointer in `PAI/Algorithm/LATEST` (e.g., v5.7.11)

**How it works:**
1. User makes a request
2. Algorithm enters OBSERVE phase
3. Builds verifiable criteria (ISC) for what "done" means
4. Moves through phases systematically
5. Verifies each phase before transitioning
6. Learns from the execution

**Example flow:**

```
Request: "Build a deployment pipeline for my web app"

OBSERVE:  ISC = { deploys on commit, runs tests, rollback on failure }
THINK:    GitHub Actions vs Jenkins vs Cloudflare → choose GitHub Actions
PLAN:     Workflow file structure, test integration, notification setup
BUILD:    Write .github/workflows/deploy.yml
EXECUTE:  Commit workflow, trigger test run
VERIFY:   ✓ Deploys successfully, ✓ Tests run, ✓ Rollback tested
LEARN:    Captured workflow pattern to Skills/Deployment/
```

### 2. Skills System

Skills are PAI's capability units. Each skill is a SKILL.md file defining:
- When to trigger (based on user request)
- Workflows (step-by-step procedures)
- Tools to use
- Expected outputs

**Structure:**

```
~/.claude/skills/
├── Art/                      # Visual content creation
├── BitterPillEngineering/    # Over-prompting detection
├── Cloudflare/               # Deployment workflows
├── Delegation/               # Multi-agent orchestration
├── ExtractWisdom/            # Content analysis
├── Fabric/                   # 240+ prompt patterns
├── Knowledge/                # Knowledge graph management
├── Research/                 # Web research and synthesis
├── Security/                 # Security assessments
├── Think/                    # Multi-model debate
└── ... 112 more
```

**Categories (12):**
- Code & Development
- Content & Media
- Research & Analysis
- Security
- Thinking & Analysis
- Utilities
- Writing & Documentation
- Business
- Personal
- Infrastructure
- Social
- Custom

**How skills load:**
- Indexed at startup via `GenerateSkillIndex.ts`
- Loaded on-demand when trigger matches request
- Can invoke other skills (composability)

### 3. Hooks System

Hooks intercept lifecycle events and run custom logic.

**Hook Points:**

```
SessionStart      → Runs when session begins
UserPromptSubmit  → Before processing user request
PreToolUse        → Before any tool executes
PostToolUse       → After tool completes
Stop              → Before response stops
SessionEnd        → When session closes
```

**Example hooks:**

```javascript
// SessionStart hook
hooks/SessionInit.hook.ts
  → Loads startup context
  → Checks for cross-session handoffs
  → Validates environment

// PreToolUse hook
hooks/SecurityValidator.hook.ts
  → Checks permissions before Write/Bash
  → Prevents path traversal
  → Scans for secrets in output

// Stop hook
hooks/VoiceAnnounce.hook.ts
  → Sends TTS notification
  → Updates status line
```

**Configuration:**

Hooks registered in `settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      { "command": "bun $PAI_DIR/hooks/SessionInit.hook.ts" }
    ],
    "PreToolUse": [
      { "command": "bun $PAI_DIR/hooks/SecurityValidator.hook.ts" }
    ]
  }
}
```

### 4. Memory System

Five-layer persistent memory architecture:

```
Layer 1: Claude Code Native
  ~/.claude/projects/
  → Session transcripts, auto-saved by Claude Code

Layer 2: ICM (Infinite Context Memory)
  MCP servers: icm, pai-memory
  → Cross-session semantic memory
  → Queryable vector embeddings

Layer 3: Learning Files
  ~/MEMORY/LEARNING/
  → 3,820+ reflection files
  → Failure patterns
  → Algorithm insights

Layer 4: Entity Graph
  Substrate + custom entities
  → 1,936 entities, 93K edges
  → People, companies, concepts, relationships

Layer 5: Work Artifacts
  ~/MEMORY/WORK/
  → PRDs (Product Requirement Documents)
  → Session artifacts
  → Project state
```

**Memory Flows:**

```
User request
    ↓
Session transcript (Layer 1)
    ↓
Semantic extraction → ICM (Layer 2)
    ↓
Learning signals → LEARNING/ (Layer 3)
    ↓
Entity extraction → Graph (Layer 4)
    ↓
Work artifacts → WORK/ (Layer 5)
```

### 5. Tools Directory

464 utilities in `PAI/Tools/`:

**Categories:**

- **Build Tools**: `BuildCLAUDE.ts`, `GenerateSkillIndex.ts`
- **Memory**: `Memory.ts`, `MemoryIngest.ts`, `KnowledgeHarvester.ts`
- **Monitoring**: `HealthCheck.ts`, `SystemAudit.ts`, `StackMonitor.ts`
- **Infrastructure**: `OmniPulse/`, `VoiceServer.ts`, `Conductor.ts`
- **Analysis**: `Inference.ts`, `ModelRouter.ts`, `BenchmarkLLM.ts`
- **Backup**: `NasBackup.sh`, `StackBackup.ts`
- **Security**: `SecurityFilter.ts`, `SecretScan.ts`

**Key Tools:**

```bash
# CLAUDE.md generation from template
bun PAI/Tools/BuildCLAUDE.ts

# AI inference with model routing
bun PAI/Tools/Inference.ts --task classification "Is this urgent?"

# System health check
bun PAI/Tools/HealthCheck.ts

# Memory operations
bun PAI/Tools/Memory.ts status
bun PAI/Tools/Memory.ts query "security assessments"

# Session progress tracking
bun PAI/Tools/SessionProgress.ts
```

### 6. Agent System

25 specialist agent types for delegation:

**Built-in Agents:**
- **Algorithm**: Full 7-phase execution
- **Engineer**: Code implementation
- **Architect**: System design
- **Researcher**: Web research and synthesis
- **Designer**: UI/UX design work
- **Security**: Security assessments
- **Writer**: Content creation (you're reading Emma's work now)
- **Reviewer**: Code review and quality
- **Planner**: Project planning

**Agent Orchestration:**

```
Delegation Skill
    ↓
Task decomposition
    ↓
Agent spawn (parallel)
    ↓
    ├─► Agent A (Engineer) → Subtask 1
    ├─► Agent B (Researcher) → Subtask 2
    └─► Agent C (Writer) → Subtask 3
    ↓
Results aggregation
    ↓
Synthesis
```

**Agent Teams:**

For coordinated work requiring state sharing:

```typescript
// Create team
TeamCreate({ name: "feature-dev", agents: ["engineer", "reviewer"] })

// Assign task
TaskCreate({ team: "feature-dev", task: "Build auth system" })

// Agents collaborate via message passing
SendMessage({ from: "engineer", to: "reviewer", content: "PR ready" })
```

## Data Flow

### Request Processing Flow

```
1. User Input
   "Research AI security and write a summary"
   
2. UserPromptSubmit Hook
   → Mode classifier: ALGORITHM mode
   → Loads Algorithm v5.7.11
   
3. Algorithm OBSERVE
   → Builds ISC: { research findings, 500-word summary, citations }
   → Checks for relevant skills: Research, Writer
   
4. Algorithm THINK
   → Strategy: Use Research skill → Writer agent
   
5. Algorithm PLAN
   → Step 1: Research skill (Antigravity engine)
   → Step 2: Writer agent (sonnet model)
   
6. Algorithm BUILD
   → Skill(Research) → web fan-out, content aggregation
   → Agent(Writer) → synthesis into narrative
   
7. Algorithm EXECUTE
   → Runs Research → outputs findings to MEMORY/RESEARCH/
   → Runs Writer → generates summary
   
8. Algorithm VERIFY
   → Check ISC: ✓ findings present, ✓ 500 words, ✓ citations
   
9. Algorithm LEARN
   → Reflection: "Research+Writer pattern effective"
   → Saves to LEARNING/

10. Stop Hook
    → Voice announcement
    → Updates status line
    
11. SessionEnd Hook
    → Memory promotion
    → Cross-session handoff check
```

## Configuration Architecture

### Single Source of Truth: settings.json

```
settings.json
    ├─► Identity
    │   ├─ principal (you)
    │   └─ daidentity (AI assistant)
    │
    ├─► Environment
    │   ├─ PAI_DIR, MEMORY_DIR
    │   ├─ OLLAMA_HOST
    │   └─ API keys path
    │
    ├─► Permissions
    │   ├─ allow rules
    │   └─ deny rules
    │
    ├─► Hooks
    │   ├─ SessionStart
    │   ├─ PreToolUse
    │   └─ ... 111 total
    │
    ├─► Context Loading
    │   ├─ contextFiles (force-loaded)
    │   └─ dynamicContext (conditional)
    │
    └─► Services
        ├─ MCP servers
        └─ Plugins
```

### Template System

```
CLAUDE.md.template
    ↓ (BuildCLAUDE.ts reads)
    ↓ (Substitutes variables from settings.json)
    ↓ (Inserts Algorithm version from LATEST)
    ↓
CLAUDE.md (generated)
    ↓
Loaded by Claude Code
```

**Variables in template:**

```markdown
{PRINCIPAL.NAME}         → Your name from settings.json
{DA.NAME}                → AI assistant name
{ALGORITHM.VERSION}      → Current Algorithm version
{SKILLS.COUNT}           → Number of loaded skills
```

## Service Architecture

### OmniPulse (Central Daemon)

```
OmniPulse (:31338)
    │
    ├─► Health Checks
    │   └─ Monitors all PAI services
    │
    ├─► Hook Execution
    │   └─ Runs async hooks
    │
    ├─► Dashboard
    │   └─ Web UI for status
    │
    └─► Agent Coordination
        └─ Multi-agent state
```

### Voice Server

```
Voice Server (:8888)
    │
    ├─► /notify endpoint
    │   └─ Receives TTS requests
    │
    ├─► Chatterbox TTS
    │   └─ Converts text to speech
    │
    └─► Audio output
        └─ System speakers
```

### Memory MCP

```
Memory MCP (unix socket)
    │
    ├─► Store
    │   └─ Persist key-value pairs
    │
    ├─► Query
    │   └─ Semantic search
    │
    └─► Embed
        └─ Vector embeddings
```

## Multi-Engine Coordination

PAI runs on three AI engines under one identity:

```
PAI Nova (Single DA Identity)
    │
    ├─► Claude Code (PNC)
    │   └─ Primary DA, full Algorithm, owns memory
    │
    ├─► Antigravity (PNG)
    │   └─ Research, web fan-out, fast factual lookup
    │
    └─► Ollama (PNO)
        └─ Local inference, zero-cost classification
```

**Cross-Engine Handoffs:**

```typescript
// PNC hands off to PNG for research
RecordRouter.ts → create handoff
    ↓
PNG receives via RecordRouter check
    ↓
PNG completes research
    ↓
RecordRouter.ts → mark complete
    ↓
PNC retrieves results
```

## Extension Points

### Adding a Skill

```bash
# Use CreateSkill skill
Skill(CreateSkill, { name: "MySkill", category: "Utilities" })

# Or manually
mkdir ~/.claude/skills/MySkill
nano ~/.claude/skills/MySkill/SKILL.md

# Regenerate index
bun ~/.claude/PAI/Tools/GenerateSkillIndex.ts
```

### Adding a Hook

```bash
# Create handler
nano ~/.claude/hooks/handlers/MyHook.hook.ts

# Register in settings.json
{
  "hooks": {
    "UserPromptSubmit": [
      { "command": "bun hooks/handlers/MyHook.hook.ts" }
    ]
  }
}
```

### Adding a Tool

```bash
# Create tool
nano ~/.claude/PAI/Tools/MyTool.ts

# Make executable
chmod +x ~/.claude/PAI/Tools/MyTool.ts

# Test
bun ~/.claude/PAI/Tools/MyTool.ts
```

### Adding User Context

```bash
# Add to USER directory
nano ~/.claude/PAI/USER/MY_CONTEXT.md

# Load at startup (optional)
# Edit settings.json → contextFiles
```

## Scaling Patterns

### Single User (Default)

- One principal
- Local Ollama for inference
- Cloud models for complex reasoning
- ~$10-50/month API costs

### Power User

- Multiple projects
- Full local model suite
- GPU for inference
- Minimal API costs (~$5-10/month)
- NAS backup

### Homelab (Reference: pai-primary)

- Proxmox host
- Multiple VMs (PAI, services, backup)
- Local GPU server (Ollama, vLLM)
- Automated backups to NAS
- ~$0-5/month API costs (mostly local)

### Team (Future)

- Shared PAI instance
- Multi-user auth
- Shared memory with RBAC
- Team skills and workflows

## Security Boundaries

### Permission System

```
settings.json permissions
    ↓
PreToolUse hook (SecurityValidator)
    ↓
Check allow/deny rules
    ↓
    ├─► Allowed → Execute tool
    └─► Denied → Block + log + notify user
```

### Secret Management

```
secrets.env (chmod 600)
    ↓
Loaded by services only
    ↓
Never logged, never in responses
    ↓
PreToolUse hook scans output
    ↓
    ├─► Secret detected → Redact + warn
    └─► Clean → Allow
```

## Performance Characteristics

### Startup Time

- Cold start: ~2-5 seconds
- Warm start: <1 second
- Skill index load: ~500ms
- Hook registration: ~200ms

### Memory Usage

- Base: ~200-500MB (Bun runtime + Claude Code)
- With Ollama models loaded: +2-8GB
- Memory files: ~100MB per 1000 sessions

### Response Times

- Simple query: <1s
- ALGORITHM mode (Extended): 30s-5min
- Multi-agent delegation: 2-15min
- Full research workflow: 5-30min

---

## Learning the Architecture

**Start here:**
1. Read `PAI/README.md` (overview)
2. Review `settings.json` (your configuration)
3. Explore `PAI/Algorithm/LATEST` (current Algorithm)
4. Browse `skills/` (capabilities)
5. Check `PAI/Tools/` (utilities)

**Understand by doing:**
- Make a request → watch the Algorithm phases
- Ask "How did you do that?" → Nova explains the flow
- Check `MEMORY/WORK/` → see artifacts
- Review hooks in `settings.json` → trace lifecycle

**Deep dive:**
- [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md) - How to install
- [CONFIGURATION.md](CONFIGURATION.md) - All configuration options
- [TROUBLESHOOTING_METHOD.md](TROUBLESHOOTING_METHOD.md) - Debug approach

PAI's architecture is designed to be understandable. Every piece has a reason, and the system explains itself as you use it.
