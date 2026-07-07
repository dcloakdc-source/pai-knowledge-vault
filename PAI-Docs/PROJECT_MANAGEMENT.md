# Project Management — Multi-Project State System

**Built:** 2026-06-30  
**Owner:** Duane  
**Engine:** PNK (OpenCode)

---

## Philosophy

**Core principle:** Unfinished projects are exploration, not failure.

For polymaths like Duane, context-switching between concurrent projects is a **feature**, not a bug. This system removes guilt from parking projects and preserves learnings even from incomplete work.

**Pattern Weaver Mind research shows:**
- Multiple concurrent explorations are natural for polymaths
- Context-switching is cognitively enriching
- "Unfinished" projects contribute to overall learning
- Parking without guilt enables bold exploration

---

## Quick Start

```bash
# List all projects
bun ~/.claude/PAI/Tools/ProjectManager.ts list

# Switch to a project
bun ~/.claude/PAI/Tools/ProjectManager.ts switch my-auth-service

# Park a project (with reason and learnings)
bun ~/.claude/PAI/Tools/ProjectManager.ts park my-blog \
  "Exploring static site options" \
  "Hugo is fast" \
  "Need better theme system" \
  "MDX would be nice"

# Resume a parked project
bun ~/.claude/PAI/Tools/ProjectManager.ts resume my-blog

# Show active project
bun ~/.claude/PAI/Tools/ProjectManager.ts active

# Archive completed/abandoned project
bun ~/.claude/PAI/Tools/ProjectManager.ts archive old-project
```

---

## Commands

### `switch <project>`

**Switch active project and load full context.**

```bash
bun ~/.claude/PAI/Tools/ProjectManager.ts switch my-auth-service
```

**Output:**
```
🔄 SWITCHED TO: my-auth-service

Path: /home/duane/projects/my-auth-service
Status: active
Tech: TypeScript, Cloudflare Workers, D1

📊 CURRENT STATE:
Phase: implementation
Focus: JWT token validation

⏭️  NEXT STEPS:
  1. Write auth integration tests
  2. Deploy to staging
  3. Performance benchmarking

✅ Active project set to: my-auth-service
```

**What it does:**
1. Updates last accessed timestamp
2. Sets as active project in registry
3. Loads `.project/state.json`
4. Shows current phase, focus, blockers, next steps
5. If previously parked, shows parking context (reason, learnings)

---

### `park <project> <reason> [learnings...]`

**Save project state and park it — no guilt.**

```bash
bun ~/.claude/PAI/Tools/ProjectManager.ts park my-blog \
  "Exploring static site generators" \
  "Hugo build time is impressive" \
  "Need to research theme customization" \
  "Content structure should be data-driven"
```

**Output:**
```
🅿️  PARKED: my-blog

Reason: Exploring static site generators

💡 LEARNINGS CAPTURED:
  1. Hugo build time is impressive
  2. Need to research theme customization
  3. Content structure should be data-driven

✅ Project parked. No guilt — exploration is valuable!
```

**What it does:**
1. Captures current project state (phase, focus, next steps, blockers)
2. Records **why** project was started (exploration goal)
3. Captures **learnings** even if incomplete
4. Saves parking entry to `~/.claude/PAI/MEMORY/STATE/parking-lot/<project>.json`
5. Updates project status to "parked" in registry
6. Clears active project if it was this one

**Philosophy:** Parking preserves context and learnings. It's not abandonment — it's strategic pausing.

---

### `resume <project>`

**Restore full context from parked project.**

```bash
bun ~/.claude/PAI/Tools/ProjectManager.ts resume my-blog
```

**Output:**
```
🔄 SWITCHED TO: my-blog

Path: /home/duane/projects/my-blog
Status: active (was parked)
Tech: Hugo, Markdown

📊 CURRENT STATE:
Phase: exploration
Focus: Theme system research

📝 PARKED CONTEXT:
Reason: Exploring static site generators

💡 LEARNINGS:
  1. Hugo build time is impressive
  2. Need to research theme customization
  3. Content structure should be data-driven

🔍 RESUME HINTS:
  1. Load ISA: ISA.md
  2. Last focus: Theme system research

🔄 Context fully restored. Ready to continue!
```

**What it does:**
1. Changes status from "parked" to "active"
2. Loads parking lot entry
3. Displays **why** project was started
4. Shows **learnings** captured when parked
5. Shows **resume hints** (ISA, last focus, etc.)
6. Sets as active project

---

### `list`

**Show all projects grouped by status.**

```bash
bun ~/.claude/PAI/Tools/ProjectManager.ts list
```

**Output:**
```
📊 PROJECT OVERVIEW

🎯 Currently active: my-auth-service

✨ ACTIVE (3):
  → my-auth-service [TypeScript, Cloudflare Workers, D1]
    OAuth 2.0 authentication service
    my-blog [Hugo, Markdown]
    Polymath PAI extensions [TypeScript]

🅿️  PARKED (2):
    static-site-research [Hugo, Jekyll, Eleventy]
    ai-training-pipeline [Python, PyTorch]

📦 ARCHIVED (1):
    old-prototype
```

**Status meanings:**
- **active** — Currently being worked on or available to switch to
- **parked** — Paused with context preserved, ready to resume
- **archived** — Completed or abandoned, state preserved for reference

---

### `active`

**Show current active project.**

```bash
bun ~/.claude/PAI/Tools/ProjectManager.ts active
```

**Output:**
```
🎯 ACTIVE PROJECT: my-auth-service

Path: /home/duane/projects/my-auth-service
Status: active
Tech: TypeScript, Cloudflare Workers, D1

OAuth 2.0 authentication service
```

---

### `archive <project>`

**Archive completed or abandoned project.**

```bash
bun ~/.claude/PAI/Tools/ProjectManager.ts archive old-prototype
```

**Output:**
```
📦 ARCHIVED: old-prototype
✅ Project archived. State preserved for future reference.
```

**What it does:**
1. Changes status to "archived"
2. Clears active project if it was this one
3. Keeps all state/parking data for reference

---

### `register <name> <path> [tech...]`

**Manually register a new project.**

```bash
bun ~/.claude/PAI/Tools/ProjectManager.ts register \
  my-project \
  /home/duane/projects/my-project \
  TypeScript React "Cloudflare Pages"
```

**Output:**
```
✅ Registered project: my-project
```

**Note:** Most projects auto-register when you initialize ProjectState via the skill. This command is for manual registration.

---

## Storage Structure

```
~/.claude/PAI/MEMORY/STATE/
├── projects-registry.json              # Global registry
└── parking-lot/
    ├── my-blog.json                    # Parking entry
    ├── ai-training-pipeline.json
    └── static-site-research.json

<project-root>/
└── .project/                           # Per-project state (via ProjectState skill)
    ├── project.json
    ├── state.json
    └── history/
```

### `projects-registry.json`

**Global registry of all projects:**

```json
{
  "projects": {
    "my-auth-service": {
      "name": "my-auth-service",
      "path": "/home/duane/projects/my-auth-service",
      "created": "2026-06-20T10:00:00Z",
      "last_accessed": "2026-06-30T14:30:00Z",
      "status": "active",
      "tech_stack": ["TypeScript", "Cloudflare Workers", "D1"],
      "description": "OAuth 2.0 authentication service",
      "isa_path": "ISA.md"
    }
  },
  "active_project": "my-auth-service",
  "updated": "2026-06-30T14:30:00Z"
}
```

### Parking Lot Entry

**`~/.claude/PAI/MEMORY/STATE/parking-lot/my-blog.json`:**

```json
{
  "project_name": "my-blog",
  "parked_at": "2026-06-25T16:00:00Z",
  "parked_by_engine": "PNK",
  "reason": "Exploring static site generators",
  "learnings": [
    "Hugo build time is impressive",
    "Need to research theme customization",
    "Content structure should be data-driven"
  ],
  "context_snapshot": {
    "phase": "exploration",
    "focus": "Theme system research",
    "next_steps": [
      "Evaluate Hugo themes",
      "Compare with Jekyll"
    ],
    "blockers": []
  },
  "resume_hints": [
    "Load ISA: ISA.md",
    "Last focus: Theme system research"
  ]
}
```

---

## Integration with ProjectState Skill

**This system extends the ProjectState skill** (`.opencode/skills/ProjectState/SKILL.md`).

**Division of responsibility:**

| Component | Purpose | Storage |
|-----------|---------|---------|
| **ProjectState skill** | Per-project state (phase, focus, blockers, history) | `<project>/.project/` |
| **ProjectManager tool** | Multi-project switching, parking lot, global registry | `~/.claude/PAI/MEMORY/STATE/` |

**Workflow:**

```bash
# 1. Initialize project state (ProjectState skill)
# In project directory:
bun ~/.claude/PAI/Tools/project-state.ts init my-project TypeScript React

# 2. Register in ProjectManager (automatic if using skills)
bun ~/.claude/PAI/Tools/ProjectManager.ts register \
  my-project \
  $(pwd) \
  TypeScript React

# 3. Work on project...

# 4. Park project when exploring elsewhere
bun ~/.claude/PAI/Tools/ProjectManager.ts park my-project \
  "Exploring React Server Components" \
  "RSC reduces client bundle" \
  "Need to test with Cloudflare"

# 5. Switch to another project
bun ~/.claude/PAI/Tools/ProjectManager.ts switch other-project

# 6. Resume original project later
bun ~/.claude/PAI/Tools/ProjectManager.ts resume my-project
```

---

## Agent Integration

**PAI agents can use ProjectManager for context switching:**

### In OBSERVE Phase

```typescript
// Check active project
const { execSync } = require("child_process");
const active = execSync(
  "bun ~/.claude/PAI/Tools/ProjectManager.ts active",
  { encoding: "utf-8" }
);

// Load project state if working on a known project
if (active.includes("my-auth-service")) {
  // Load ISA, state, etc.
}
```

### When Parking

```typescript
// Park current project with learnings
execSync(`
  bun ~/.claude/PAI/Tools/ProjectManager.ts park my-project \
    "Explored authentication patterns" \
    "Passkeys require HTTPS" \
    "Need to test across browsers"
`);
```

### When Switching

```typescript
// Switch projects
execSync("bun ~/.claude/PAI/Tools/ProjectManager.ts switch other-project");

// Context automatically loaded by ProjectManager
```

---

## Use Cases

### 1. Morning Context Recovery

**Scenario:** Duane worked on 3 projects yesterday. Which one should he resume?

```bash
# Show all projects
bun ~/.claude/PAI/Tools/ProjectManager.ts list

# Pick one
bun ~/.claude/PAI/Tools/ProjectManager.ts resume my-auth-service
```

**Output shows:**
- What was being worked on
- Why project was started
- Key learnings captured
- Next steps

**Decision made in <30 seconds.**

---

### 2. Exploration Without Guilt

**Scenario:** Duane wants to explore static site generators but has 2 active projects.

```bash
# Park current project with reason
bun ~/.claude/PAI/Tools/ProjectManager.ts park my-auth-service \
  "Reached blocker: waiting on OAuth provider" \
  "JWT validation works" \
  "Tests are solid"

# Start new exploration
mkdir ~/projects/static-site-research
cd ~/projects/static-site-research

# Work on new thing...

# Later: Park exploration with learnings
bun ~/.claude/PAI/Tools/ProjectManager.ts park static-site-research \
  "Evaluating Hugo vs Jekyll vs Eleventy" \
  "Hugo is fastest" \
  "Jekyll has best themes" \
  "Eleventy is most flexible"

# Resume original project
bun ~/.claude/PAI/Tools/ProjectManager.ts resume my-auth-service
```

**Result:** Both projects preserved. Learnings captured. No guilt. Can revisit static-site research anytime.

---

### 3. Multi-Day Projects

**Scenario:** Working on large project across multiple sessions.

```bash
# Day 1: Start work
bun ~/.claude/PAI/Tools/ProjectManager.ts switch big-project

# Day 1: End of session (auto-capture via ProjectState)
bun ~/.claude/PAI/Tools/project-state.ts update \
  --phase implementation \
  --focus "User authentication"

# Day 2: Resume
bun ~/.claude/PAI/Tools/ProjectManager.ts switch big-project
# → Shows yesterday's focus, next steps, blockers
```

**ProjectState handles session state. ProjectManager handles project switching.**

---

## Success Metrics

| Metric | Target | Current |
|--------|--------|---------|
| Context recovery time | <30s | ✅ (list → switch → ready) |
| Parked projects with learnings | 100% | ✅ (required field) |
| Guilt-free parking | Subjective | ✅ (messaging emphasizes value) |
| Multi-session continuity | 100% | ✅ (state + parking preserved) |
| Project count supported | 10+ concurrent | ✅ (no limit) |

---

## Future Enhancements

### Planned

- [ ] **Auto-parking on inactivity** — If project untouched for 7 days, prompt to park with reason
- [ ] **Project templates** — Initialize with common patterns (web app, CLI tool, research, etc.)
- [ ] **Cross-project search** — `projectmanager search "authentication"` → shows relevant context across all projects
- [ ] **Timeline view** — Visual timeline of project switches and learnings
- [ ] **Knowledge linking** — Link parking entries to Knowledge Archive notes

### Considered

- [ ] **Effort estimation** — Track time per project, surface patterns
- [ ] **Dependency tracking** — "Project A blocked by Project B"
- [ ] **Team integration** — Shared project registry for multi-person PAI setups

---

## Testing

**Run tests:**

```bash
cd ~/.claude/PAI/Tools
bun test __tests__/ProjectManager.test.ts
```

**Test coverage:**
- Registry operations (register, update, set active)
- Project state loading
- Parking lot operations
- Full workflow integration

---

## Troubleshooting

### "Project not found" error

**Cause:** Project not in registry.

**Fix:**
```bash
bun ~/.claude/PAI/Tools/ProjectManager.ts register \
  my-project \
  /path/to/project \
  TypeScript
```

---

### Parking entry not showing on switch

**Cause:** Project never parked, or parking file deleted.

**Fix:** Park the project with current context:
```bash
bun ~/.claude/PAI/Tools/ProjectManager.ts park my-project \
  "Reason here" \
  "Learning 1" \
  "Learning 2"
```

---

### Can't load project state

**Cause:** `.project/` directory missing (not using ProjectState skill).

**Fix:** Initialize ProjectState:
```bash
cd /path/to/project
bun ~/.claude/PAI/Tools/project-state.ts init my-project TypeScript
```

---

## Related Documentation

- `skills/ProjectState/SKILL.md` — Per-project state management
- `PAI/USER/PRINCIPAL_IDENTITY.md` — Duane's working patterns
- `PAI/DOCUMENTATION/MEMORY_SYSTEMS.md` — Cross-engine memory architecture

---

**Last Updated:** 2026-06-30  
**Status:** ✅ Production Ready  
**Tests:** ✅ Passing
