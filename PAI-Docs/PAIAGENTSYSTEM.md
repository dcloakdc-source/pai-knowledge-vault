# PAI Agent System

**Authoritative reference for agent routing in PAI. Three distinct systems exist—never confuse them.**

---

## 🚨 THREE AGENT SYSTEMS — CRITICAL DISTINCTION

PAI has three agent systems that serve different purposes. Confusing them causes routing failures.

| System | What It Is | When to Use | Has Unique Voice? |
|--------|-----------|-------------|-------------------|
| **Task Tool Subagent Types** | Pre-built agents in Claude Code (Architect, Designer, Engineer, Explore, etc.) | Internal workflow use ONLY | No |
| **Named Agents** | Persistent identities with backstories and ElevenLabs voices (Serena, Marcus, Rook, etc.) | Recurring work, voice output, relationships | Yes |
| **Custom Agents** | Dynamic agents composed via ComposeAgent from traits | When user says "custom agents" | Yes (trait-mapped) |

---

## 🚫 FORBIDDEN PATTERNS

**When user says "custom agents":**

```typescript
// ❌ WRONG - These are Task tool subagent_types, NOT custom agents
Task({ subagent_type: "Architect", prompt: "..." })
Task({ subagent_type: "Designer", prompt: "..." })
Task({ subagent_type: "Engineer", prompt: "..." })

// ✅ RIGHT - Invoke the Agents skill for custom agents
Skill("Agents")  // → CreateCustomAgent workflow
// OR follow the workflow directly:
// 1. Run ComposeAgent with different trait combinations
// 2. Launch agents with the generated prompts
// 3. Each gets unique personality + voice
```

---

## Routing Rules

### The Word "Custom" Is the Trigger

| User Says | Action | Implementation |
|-----------|--------|----------------|
| "**custom agents**", "spin up **custom** agents" | Invoke Agents skill | `Skill("Agents")` → CreateCustomAgent workflow |
| "agents", "launch agents", "parallel agents" | Custom agents via Agents skill | `Skill("Agents")` → ComposeAgent → `Task({ subagent_type: "general-purpose" })` |
| "research X", "investigate Y" | Research skill | `Skill("Research")` → appropriate researcher agents |
| "use Remy", "get Ava to" | Named agent | Use appropriate researcher subagent_type |
| (Code implementation) | Engineer | `Task({ subagent_type: "Engineer" })` |
| (Architecture/design) | Architect | `Task({ subagent_type: "Architect" })` |

### Custom Agent Creation Flow

When user requests custom agents:

1. **Invoke Agents skill** via `Skill("Agents")` or follow CreateCustomAgent workflow
2. **Run ComposeAgent** for EACH agent with DIFFERENT trait combinations
3. **Extract prompt and voice_id** from ComposeAgent output
4. **Launch agents** with Task tool using the composed prompts
5. **Voice results** using each agent's unique voice_id

```bash
# Example: 3 custom research agents
bun run ~/.claude/skills/Agents/Tools/ComposeAgent.ts --traits "research,enthusiastic,exploratory"
bun run ~/.claude/skills/Agents/Tools/ComposeAgent.ts --traits "research,skeptical,systematic"
bun run ~/.claude/skills/Agents/Tools/ComposeAgent.ts --traits "research,analytical,synthesizing"
```

---

## Task Tool Subagent Types (Internal Use Only)

These are pre-built agents in the Claude Code Task tool. They are for **internal workflow use**, not for user-requested "custom agents."

| Subagent Type | Purpose | When Used |
|---------------|---------|-----------|
| `Architect` | System design | Development skill workflows |
| `Designer` | UX/UI design | Development skill workflows |
| `Engineer` | Code implementation | Development skill workflows |
| `general-purpose` | Custom agents via ComposeAgent | Parallel work with task-specific prompts |
| `Explore` | Codebase exploration | Finding files, understanding structure |
| `Plan` | Implementation planning | Plan mode |
| `QATester` | Quality assurance | Browser testing workflows |
| `Pentester` | Security testing | WebAssessment workflows |
| `ClaudeResearcher` | Claude-based research | Research skill workflows |
| `GeminiResearcher` | Gemini-based research | Research skill workflows |
| `GrokResearcher` | Grok-based research | Research skill workflows |

**These do NOT have unique voices or ComposeAgent composition.**

---

## Named Agents (Persistent Identities)

Named agents have rich backstories, personality traits, and mapped ElevenLabs voices. They provide relationship continuity across sessions.

| Agent | Role | Voice | Use For |
|-------|------|-------|---------|
| Serena Blackwood | Architect | Premium UK Female | Long-term architecture decisions |
| Marcus Webb | Engineer | Premium Male | Strategic technical leadership |
| Rook Blackburn | Pentester | Enhanced UK Male | Security testing with personality |
| Ava Sterling | Claude Researcher | Premium US Female | Strategic research |
| Alex Rivera | Gemini Researcher | Multi-perspective | Comprehensive analysis |

**Full backstories and voice settings:** Individual `agents/*.md` files (persona frontmatter + body)

---

## Custom Agents (Dynamic Composition)

Custom agents are composed on-the-fly from traits using ComposeAgent. Each unique trait combination maps to a different ElevenLabs voice.

### Trait Categories

**Expertise** (domain knowledge):
`security`, `legal`, `finance`, `medical`, `technical`, `research`, `creative`, `business`, `data`, `communications`

**Personality** (behavior style):
`skeptical`, `enthusiastic`, `cautious`, `bold`, `analytical`, `creative`, `empathetic`, `contrarian`, `pragmatic`, `meticulous`

**Approach** (work style):
`thorough`, `rapid`, `systematic`, `exploratory`, `comparative`, `synthesizing`, `adversarial`, `consultative`

### Voice Mapping Examples

| Trait Combo | Voice | Why |
|-------------|-------|-----|
| contrarian + skeptical | Clyde (gravelly) | Challenging intensity |
| enthusiastic + creative | Jeremy (energetic) | High-energy creativity |
| security + adversarial | Callum (edgy) | Hacker character |
| analytical + meticulous | Charlotte (sophisticated) | Precision analysis |

**Full trait definitions and voice mappings:** `skills/Agents/Data/Traits.yaml`

---

## Model Selection

Always specify the appropriate model for agent work:

| Task Type | Model | Speed |
|-----------|-------|-------|
| Simple checks, grunt work | `haiku` | 10-20x faster |
| Standard analysis, implementation | `sonnet` | Balanced |
| Deep reasoning, architecture | `opus` | Maximum intelligence |

```typescript
// Parallel custom agents benefit from haiku/sonnet for speed
Task({ prompt: agentPrompt, subagent_type: "general-purpose", model: "sonnet" })
```

---

## Spotcheck Pattern

**Always launch a spotcheck agent after parallel work:**

```typescript
Task({
  prompt: "Verify consistency across all agent outputs: [results]",
  subagent_type: "general-purpose",
  model: "haiku"
})
```

---

## Agent Identity (WIMSE)

Every PAI agent has a stable, globally unique WIMSE/SPIFFE URI. This is the **canonical identity** for each agent — used in delegation chains, audit trails, and future zeroid integration (Tier 1 of highflame-ai/zeroid).

**URI format:** `spiffe://pai.local/agent/{slug}` — personal infrastructure scope, not public-facing.

### External CLI Agents

| Agent | ID | WIMSE URI |
|-------|----|-----------|
| PAI Nova Claude (me) | AGT-001 | `spiffe://pai.local/agent/pai-nova-claude` |
| PAI Nova Gemini | AGT-002 | `spiffe://pai.local/agent/pai-nova-gemini` |
| PAI Nova OpenCode | AGT-003 | `spiffe://pai.local/agent/pai-nova-opencode` |

### Task Tool Subagents

| Agent | Persona | WIMSE URI |
|-------|---------|-----------|
| Architect | Serena Blackwood | `spiffe://pai.local/agent/serena-blackwood` |
| Engineer | Marcus Webb | `spiffe://pai.local/agent/marcus-webb` |
| Pentester | Rook Blackburn | `spiffe://pai.local/agent/rook-blackburn` |
| ClaudeResearcher | Ava Sterling | `spiffe://pai.local/agent/ava-sterling` |
| GeminiResearcher | Alex Rivera | `spiffe://pai.local/agent/alex-rivera` |
| GrokResearcher | Johannes | `spiffe://pai.local/agent/johannes` |
| CodexResearcher | Remy | `spiffe://pai.local/agent/remy` |
| PerplexityResearcher | Ava Chen | `spiffe://pai.local/agent/ava-chen` |
| Algorithm | Vera Sterling | `spiffe://pai.local/agent/vera-sterling` |
| Artist | Priya Desai | `spiffe://pai.local/agent/priya-desai` |
| Designer | Aditi Sharma | `spiffe://pai.local/agent/aditi-sharma` |
| QATester | Quinn Torres | `spiffe://pai.local/agent/quinn-torres` |
| Intern | Dev Patel | `spiffe://pai.local/agent/dev-patel` |
| LocalAnalyst | Adam Qwen | `spiffe://pai.local/agent/adam-qwen` |
| LocalReasoner | Callum Seek | `spiffe://pai.local/agent/callum-seek` |
| LocalRunner | Eric Llama | `spiffe://pai.local/agent/eric-llama` |
| LocalSummarizer | Antoni Gemma | `spiffe://pai.local/agent/antoni-gemma` |
| CodeReviewer | — | `spiffe://pai.local/agent/code-reviewer` |
| ProductManager | — | `spiffe://pai.local/agent/product-manager` |
| BackendDev | — | `spiffe://pai.local/agent/backend-dev` |
| TeamLead | — | `spiffe://pai.local/agent/team-lead` |
| Tester | — | `spiffe://pai.local/agent/tester` |

### Delegation Model

When an agent delegates to a sub-agent, the delegation chain should be preserved in task metadata:

```json
{
  "delegator_uri": "spiffe://pai.local/agent/pai-nova-claude",
  "actor_uri": "spiffe://pai.local/agent/marcus-webb",
  "delegation_depth": 1,
  "scope": "code:write"
}
```

**Tier 2 readiness:** When zeroid (`docker compose up -d`, port 8899) is running, register agents from this table and use `client.tokens.delegate()` for cross-agent Switchboard routing.

---

## External CLI Agents

External CLI agents are separate AI runtimes invocable from PAI via Bash. They are NOT Claude Code subagents — they cannot be used with the `Task` tool. They have their own models, skill sets, and provider configurations.

| Agent | ID | Alias | Runtime | Provider | WIMSE URI | Invoke |
|-------|----|-------|---------|----------|-----------|--------|
| PAI Nova OpenCode | AGT-003 | `opai` | OpenCode v1.4.4 | Zen (free) | `spiffe://pai.local/agent/pai-nova-opencode` | `bun run .opencode/PAI/Tools/pai.ts` |
| PAI Nova Gemini | AGT-002 | `png` | Gemini CLI v0.38.0 | Google Gemini | `spiffe://pai.local/agent/pai-nova-gemini` | `gemini` |

### Invoking PAI Nova OpenCode (AGT-003)

**One-shot prompt (non-interactive):**
```bash
/home/duane/.bun/bin/bun run /PAI/Source/Projects/pai-opencode/.opencode/PAI/Tools/pai.ts prompt "your task here"
```

**Interactive session:**
```bash
opai           # via shell alias (requires sourced ~/.bashrc)
```

**From Claude Code as a Bash delegate:**
```bash
# Delegate a coding task to OpenCode's zen free models
cd /PAI/Source/Projects/pai-opencode && \
  /home/duane/.bun/bin/bun run .opencode/PAI/Tools/pai.ts prompt "implement X in project Y"
```

**Switchboard routing** (async, file-based):
Drop a task into PAI outbox with `"target_agent": "opencode"` — the SwitchboardBridge will route it to the OpenCode inbox.

### When to Use OpenCode vs Claude Code

| Use OpenCode (AGT-003) | Use Claude Code (AGT-001) |
|------------------------|--------------------------|
| Zero-cost coding tasks (zen free) | Tasks requiring Claude's reasoning depth |
| Provider-agnostic workflows | Hook system, memory, PAI infrastructure |
| OpenCode's 17 built-in agents | Claude Code Task tool subagents |
| Budget-critical periods | Standard/full-capability sessions |

---

## References

- **Agents Skill:** `skills/Agents/SKILL.md` — Custom agent creation, workflows
- **ComposeAgent:** `skills/Agents/Tools/ComposeAgent.ts` — Dynamic composition tool
- **Traits:** `skills/Agents/Data/Traits.yaml` — Trait definitions and voice mappings
- **Agent Personalities:** Individual `agents/*.md` files — Named agent backstories and voice settings

---

*Last updated: 2026-01-14*
