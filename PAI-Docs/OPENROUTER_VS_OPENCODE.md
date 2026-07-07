# OpenRouter vs OpenCode: Orchestration Comparison

**Version:** 1.0  
**Updated:** 2026-06-24  
**Author:** PAI Nova (PNK)

## Executive Summary

**OpenRouter** and **OpenCode** solve fundamentally different problems:

- **OpenRouter** is a **model proxy/router** — a unified API gateway that routes LLM requests to 400+ models across 70+ providers with automatic failover and cost optimization
- **OpenCode** is a **coding agent platform** — an interactive terminal agent that writes code, plans multi-file changes, and executes complex development workflows

**The comparison you're asking about:**
- OpenRouter is like **PAI's Inference.ts + inference-matrix.json on steroids** — centralized model routing with a commercial API
- OpenCode is a **full-featured AI coding assistant** competing with Claude Code, Cursor, Aider, GitHub Copilot

**Can they work together?** Yes. OpenRouter could be a **backend provider** for OpenCode (or PAI's inference layer), giving access to 400+ models via a single API key.

---

## Detailed Comparison

### OpenRouter: Unified LLM API Gateway

#### What It Is

OpenRouter is a **commercial API service** that provides:
1. Single API endpoint for 400+ models (Claude, GPT, Gemini, Llama, Mistral, etc.)
2. Automatic provider failover (if Anthropic is down, route to AWS Bedrock Claude)
3. Cost optimization (picks cheapest provider for a given model)
4. Usage-based pricing (no subscriptions, pay per token)
5. Data residency controls (EU-only routing, for compliance)
6. OpenAI SDK compatibility (drop-in replacement)

#### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Your Application                         │
│  (PAI, custom app, agent framework)                         │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      │ Single API call
                      │ OpenAI-compatible format
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                  OpenRouter Gateway                         │
│  • Model routing (400+ models, 70+ providers)               │
│  • Automatic failover (Anthropic → Bedrock → Vertex)        │
│  • Cost optimization (picks cheapest provider)              │
│  • Load balancing (distributes across regions)              │
│  • Data policy enforcement (no logging, GDPR compliance)    │
└─────────────────────┬───────────────────────────────────────┘
                      │
          ┌───────────┼───────────┬───────────┐
          ▼           ▼           ▼           ▼
    ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
    │Anthropic│ │ OpenAI  │ │ Google  │ │  AWS    │
    │ Claude  │ │   GPT   │ │ Gemini  │ │ Bedrock │
    └─────────┘ └─────────┘ └─────────┘ └─────────┘
```

#### What It Does

- **Model abstraction** — Call `anthropic/claude-opus-4` without caring if it hits Anthropic, AWS Bedrock, or GCP Vertex
- **Provider failover** — If Anthropic API is down, automatically retry via Bedrock Claude
- **Cost arbitrage** — Picks cheapest provider (e.g., Bedrock Claude costs less than direct Anthropic)
- **Unified billing** — One invoice for all LLM usage across providers
- **Data controls** — Route GPT to Azure OpenAI (EU-only) for GDPR compliance

#### What It Doesn't Do

- ❌ **No agent framework** — Just an API proxy, no planning/execution logic
- ❌ **No code editing** — Doesn't interact with files, git, or terminal
- ❌ **No conversation memory** — Stateless per request
- ❌ **No tool calling orchestration** — You build that yourself
- ❌ **No terminal UI** — Pure API service

#### Pricing

- Pay-per-use (no subscription)
- Prices vary by model (see [openrouter.ai/models](https://openrouter.ai/models))
- Example: `anthropic/claude-sonnet-4-6` → $3.00/M input, $15.00/M output (same as direct Anthropic)
- Cost optimization: May route to cheaper provider for same model

---

### OpenCode: Provider-Agnostic Coding Agent

#### What It Is

OpenCode is an **interactive terminal agent** that:
1. Writes and edits code across multiple files
2. Plans complex changes with dependency analysis
3. Runs tests, commits to git, creates PRs
4. Switches between LLM backends (Claude, GPT, Gemini, Ollama)
5. Provides TUI for approval, planning, file diffs
6. Runs locally with full file system access

#### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                  OpenCode Terminal UI                       │
│  • Interactive chat                                         │
│  • File tree navigation                                     │
│  • Planning pane (shows multi-step plan)                    │
│  • Approval gates (human confirms before execution)         │
│  • Git integration                                          │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      │ Agent loop
                      ▼
┌─────────────────────────────────────────────────────────────┐
│               OpenCode Agent Core                           │
│  • Planning: Break down tasks into steps                    │
│  • Execution: Edit files, run commands                      │
│  • Verification: Run tests, check results                   │
│  • Context: Track conversation, file changes, git state     │
│  • Memory: Persist sessions, resume work                    │
└─────────────────────┬───────────────────────────────────────┘
                      │
          ┌───────────┼───────────┬───────────┐
          ▼           ▼           ▼           ▼
    ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
    │ Claude  │ │   GPT   │ │ Gemini  │ │ Ollama  │
    │ Sonnet  │ │  4o     │ │  Flash  │ │ Local   │
    └─────────┘ └─────────┘ └─────────┘ └─────────┘
```

#### What It Does

- **Multi-file editing** — Plans and executes changes across many files
- **Git integration** — Commits, branches, PRs
- **Test execution** — Runs tests, interprets failures, fixes bugs
- **Planning loop** — Shows plan before execution, human approves
- **Provider switching** — Switch from Claude to GPT mid-session if needed
- **Session persistence** — Resume work across days
- **MCP integration** — Uses Model Context Protocol for tools

#### What It Doesn't Do

- ❌ **No model routing/failover** — If Claude API is down, session fails (you manually switch provider)
- ❌ **No cost optimization** — Uses whichever provider/model you configure
- ❌ **No unified billing** — You pay each provider directly
- ❌ **No API service** — Local terminal app, not a hosted service

#### Pricing

- **Free** (open source)
- You pay for LLM API usage directly (Anthropic, OpenAI, Google, etc.)
- Local models (Ollama) are free

---

## PAI's Current Model Routing vs OpenRouter

### PAI's Inference Layer (Current State)

PAI already has a **mini-OpenRouter** built-in:

#### Components

1. **Inference.ts** — Unified dispatcher for Ollama, Claude, Gemini, OpenAI, Mercury
2. **inference-matrix.json** — Routes 11 task types to primary/fallback models
3. **ModelMatrix catalog** — 20 models with capability tags, cost, context windows
4. **Ollama local** — 22 models running at 192.168.50.20:11434 (free)
5. **Cost tracking** — Records usage per session
6. **Auto-failover** — Falls back from local to cloud if Ollama fails

#### Routing Example

```typescript
// Fast classification → Ollama (free)
const result = await inference({
  taskType: 'fast_classification',
  systemPrompt: 'Classify sentiment',
  userPrompt: 'I love this product!',
});
// Routes to: ollama/qwen3:8b (local, free)
// Fallback if Ollama down: gemini-1.5-flash (cheap cloud)

// Complex reasoning → Local first, Claude fallback
const result = await inference({
  taskType: 'complex_reasoning',
  systemPrompt: 'Solve this logic puzzle',
  userPrompt: '...',
});
// Routes to: ollama/gemma2:9b (local, free)
// Fallback if quality insufficient: claude/opus (expensive, accurate)
```

#### Capabilities

✅ **Task-based routing** — 11 task types mapped to best model  
✅ **Local-first** — Ollama models preferred for cost savings  
✅ **Automatic fallback** — Cloud backup if local fails  
✅ **Cost tracking** — Per-session usage logs  
✅ **Multiple providers** — Ollama, Anthropic, Google, OpenAI, Mercury  

❌ **No provider failover** — If Anthropic API is down, fails (doesn't try Bedrock)  
❌ **No cost arbitrage** — Always uses direct provider (not cheapest route)  
❌ **Manual matrix updates** — `inference-matrix.json` is hand-edited  
❌ **No data residency** — Can't force "EU-only" routing  

### What OpenRouter Would Add

If PAI integrated OpenRouter:

#### Gains

1. **400+ models instantly** — Access entire model ecosystem via one API key
2. **Provider failover** — Anthropic down? Auto-retry Claude via Bedrock
3. **Cost arbitrage** — Picks cheapest provider for a given model
4. **Data residency** — Force GDPR-compliant routing
5. **Unified billing** — One invoice instead of 5 provider bills
6. **Less API key management** — One key instead of ANTHROPIC_API_KEY, OPENAI_API_KEY, GOOGLE_API_KEY, etc.

#### Tradeoffs

1. **New dependency** — PAI relies on OpenRouter staying online
2. **Cost markup** — OpenRouter prices = provider prices (no markup currently, but could change)
3. **Less control** — OpenRouter picks provider, you don't control routing
4. **Latency overhead** — Extra hop through OpenRouter gateway
5. **Data privacy** — Requests flow through third-party (OpenRouter's policies apply)

#### Integration Pattern

```typescript
// Option 1: Replace direct Anthropic/OpenAI calls with OpenRouter
import Anthropic from "@anthropic-ai/sdk";

const anthropic = new Anthropic({
  apiKey: process.env.OPENROUTER_API_KEY,
  baseURL: "https://openrouter.ai/api/v1",
  defaultHeaders: {
    "HTTP-Referer": "https://pai.duane.ai",
    "X-Title": "PAI Nova",
  },
});

// Now all Claude calls route through OpenRouter
const message = await anthropic.messages.create({
  model: "anthropic/claude-sonnet-4-6",
  max_tokens: 8000,
  messages: [{ role: "user", content: "Hello!" }],
});

// Option 2: Add OpenRouter as a provider in Inference.ts
// inference-matrix.json:
{
  "routing_matrix": {
    "complex_reasoning": {
      "primary": "ollama/gemma2:9b",
      "fallback": "openrouter/anthropic/claude-opus-4"  // Via OpenRouter
    }
  }
}
```

---

## OpenCode Integration with OpenRouter

### Can OpenCode Use OpenRouter as a Backend?

**Yes.** OpenCode's LLM abstraction allows swapping providers.

#### How It Works Today

OpenCode uses:
1. **Direct provider APIs** (Anthropic, OpenAI, Google)
2. **Model-specific endpoints** (configured per session)

#### Adding OpenRouter

```jsonc
// ~/.config/opencode/opencode.json
{
  "providers": [
    {
      "id": "openrouter",
      "name": "OpenRouter",
      "baseUrl": "https://openrouter.ai/api/v1",
      "apiKeyEnv": "OPENROUTER_API_KEY",
      "models": [
        "anthropic/claude-sonnet-4-6",
        "openai/gpt-4o",
        "google/gemini-1.5-pro",
        "meta-llama/llama-3.3-70b-instruct",
        // ... 400+ more
      ]
    }
  ],
  "defaultProvider": "openrouter",
  "defaultModel": "anthropic/claude-sonnet-4-6"
}
```

Now OpenCode routes all LLM calls through OpenRouter.

#### Benefits for OpenCode

1. **Model diversity** — Try 400+ models without adding provider integrations
2. **Automatic failover** — If Claude is down, OpenRouter routes to Bedrock
3. **Cost savings** — OpenRouter picks cheapest provider route
4. **Simplified config** — One API key instead of per-provider keys

#### Tradeoffs for OpenCode

1. **New dependency** — OpenCode fails if OpenRouter is down
2. **Less control** — Can't force specific provider (Anthropic direct vs Bedrock)
3. **Latency** — Extra routing hop

---

## Architecture Comparison Matrix

| Feature | OpenRouter | OpenCode | PAI Inference.ts |
|---------|-----------|----------|------------------|
| **Type** | API Gateway | Coding Agent | Local Dispatcher |
| **Providers** | 70+ (via API) | 4 (direct) | 5 (direct: Ollama, Anthropic, Google, OpenAI, Mercury) |
| **Models** | 400+ | ~20 configured | 22 Ollama + 10 cloud |
| **Routing** | Automatic (provider failover) | Manual (user picks) | Task-based (matrix.json) |
| **Failover** | ✅ Provider-level | ❌ None | ⚠️ Model-level only |
| **Cost optimization** | ✅ Picks cheapest route | ❌ None | ⚠️ Local-first (Ollama free) |
| **Data residency** | ✅ GDPR controls | ❌ None | ❌ None |
| **Latency overhead** | ~50-100ms (extra hop) | Direct | Direct (local) or cloud |
| **Pricing** | Pay-per-token | Free (you pay LLMs) | Free (you pay LLMs) |
| **API key management** | 1 key (OpenRouter) | N keys (per provider) | N keys (per provider) |
| **Code editing** | ❌ None | ✅ Full IDE-like | ❌ None |
| **Planning/execution** | ❌ None | ✅ Multi-step workflows | ❌ None |
| **Terminal UI** | ❌ None | ✅ Interactive TUI | ❌ CLI only |
| **Session persistence** | ❌ Stateless | ✅ Resume sessions | ❌ Stateless |
| **Git integration** | ❌ None | ✅ Commit, branch, PR | ❌ None |
| **Tool calling** | ⚠️ Pass-through | ✅ Native | ⚠️ Manual orchestration |
| **MCP support** | ❌ None | ✅ Full | ⚠️ Via custom tools |
| **Local models** | ❌ None (cloud only) | ✅ Ollama | ✅ Ollama (22 models) |
| **Deployment** | ☁️ SaaS | 💻 Local binary | 💻 Local script |
| **Open source** | ❌ Proprietary | ✅ Open source | ✅ PAI (internal) |

---

## Use Cases: When to Use What

### Use OpenRouter When:

1. **You need 400+ models** — Access entire LLM ecosystem without per-provider setup
2. **High availability is critical** — Automatic provider failover
3. **Cost is a concern** — Let OpenRouter pick cheapest route
4. **Data compliance** — Need GDPR-compliant routing (EU-only providers)
5. **Unified billing** — Want one invoice instead of multiple provider bills
6. **You're building an API service** — Pass-through LLM gateway for your app
7. **Quick prototyping** — Try many models without signup overhead

### Use OpenCode When:

1. **You're writing code** — Need multi-file editing, git, test execution
2. **Complex multi-step tasks** — Planning loop with human approval
3. **Provider flexibility** — Want to switch backends mid-session
4. **Local models** — Use Ollama for free inference
5. **Session persistence** — Resume work across days
6. **Open source** — Need to audit/modify the agent
7. **No external dependencies** — Run fully local (Ollama only)

### Use PAI Inference.ts When:

1. **Task-based routing** — Automatically pick best model per task type
2. **Local-first** — Prefer Ollama (free) with cloud fallback
3. **Cost tracking** — Log per-session LLM usage
4. **Integrated with PAI** — Already using PAI infrastructure
5. **Custom routing logic** — Need domain-specific model selection
6. **Benchmark-driven** — Empirically calibrate model quality per task

---

## Recommended PAI Architecture: Hybrid Approach

### Current PAI Setup (v5.1.0)

```
User
  │
  ├─ Interactive Engines
  │   ├─ PNC (Claude Code) → Direct Anthropic API
  │   ├─ PNG (Antigravity) → Direct Google API
  │   ├─ PNX (Codex) → Direct OpenAI API
  │   └─ PNK (OpenCode) → Direct Anthropic/OpenAI/Google APIs
  │
  └─ Background Services (Daemons)
      ├─ Inference.ts → Ollama (local) + Anthropic/Google/OpenAI (cloud)
      ├─ Functions API → Stateless LLM calls
      ├─ Local Council → Multi-model debate
      └─ Skill Dispatcher → Task execution
```

### Proposed: Add OpenRouter as Optional Backend

```
User
  │
  ├─ Interactive Engines (unchanged)
  │   ├─ PNC (Claude Code) → Direct Anthropic API
  │   ├─ PNG (Antigravity) → Direct Google API
  │   ├─ PNX (Codex) → Direct OpenAI API
  │   └─ PNK (OpenCode) → ✨ OPTION: OpenRouter or Direct APIs
  │
  └─ Background Services
      ├─ Inference.ts
      │   ├─ Tier 1: Ollama (local, free) ← Keep as primary
      │   ├─ Tier 2: ✨ OpenRouter (400+ models, failover) ← NEW
      │   └─ Tier 3: Direct APIs (fallback if OpenRouter down)
      │
      ├─ Functions API → ✨ OPTION: OpenRouter backend
      ├─ Local Council → ✨ OPTION: Access 400+ models via OpenRouter
      └─ Skill Dispatcher → (unchanged)
```

### Integration Steps

#### Phase 1: Passive Integration (Read-Only)

Add OpenRouter as a **fallback tier** without changing existing routing:

```typescript
// PAI/Tools/Inference.ts

async function inferenceOpenRouter(options: InferenceOptions, modelName: string): Promise<InferenceResult> {
  const apiKey = process.env.OPENROUTER_API_KEY;
  if (!apiKey) return { success: false, error: 'OPENROUTER_API_KEY missing', ...};

  const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json',
      'HTTP-Referer': 'https://pai.duane.ai',
      'X-Title': 'PAI Nova',
    },
    body: JSON.stringify({
      model: modelName,
      messages: [
        { role: 'system', content: options.systemPrompt },
        { role: 'user', content: options.userPrompt },
      ],
    }),
  });

  const result = await response.json();
  return { success: true, output: result.choices[0].message.content, ... };
}
```

```json
// PAI/Tools/inference-matrix.json
{
  "routing_matrix": {
    "complex_reasoning": {
      "primary": "ollama/gemma2:9b",
      "fallback": "claude/opus",
      "openrouter_fallback": "openrouter/anthropic/claude-opus-4"  // NEW
    }
  }
}
```

**Result:** No behavior change. OpenRouter is available but not used by default.

#### Phase 2: Active Integration (Replace Direct Calls)

For **non-interactive services** (Functions API, Local Council), route through OpenRouter:

```typescript
// PAI/Tools/FunctionsAPI/server.ts

const OPENROUTER_ENABLED = process.env.OPENROUTER_ENABLED === 'true';

if (OPENROUTER_ENABLED) {
  // Use OpenRouter for all LLM calls
  const anthropic = new Anthropic({
    apiKey: process.env.OPENROUTER_API_KEY,
    baseURL: "https://openrouter.ai/api/v1",
  });
} else {
  // Use direct Anthropic API
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });
}
```

**Result:** Background services use OpenRouter. Interactive engines (PNC, PNK) still use direct APIs for lower latency.

#### Phase 3: Optional for Interactive (User Choice)

Let user choose per session:

```bash
# Use direct APIs (current behavior)
opencode

# Use OpenRouter backend (new option)
opencode --provider openrouter

# Or configure in settings
# ~/.config/opencode/opencode.json
{
  "defaultProvider": "openrouter",  # or "anthropic" / "openai"
}
```

**Result:** User controls routing per workflow. High-latency-tolerance tasks (research, batch processing) can use OpenRouter for cost/availability. Low-latency tasks (interactive coding) use direct APIs.

---

## Cost Analysis

### Monthly PAI Usage (Estimated)

| Service | Current Provider | Monthly Cost | Via OpenRouter | Savings |
|---------|------------------|--------------|----------------|---------|
| **Interactive PNC** | Direct Anthropic | $150 | ~$150 (same) | $0 |
| **Background inference** | Ollama (free) | $0 | $0 | $0 |
| **Functions API** | Direct Anthropic | $30 | ~$30 (same) | $0 |
| **Local Council** | Multi-provider | $50 | ~$45 (arbitrage) | $5 |
| **Research (PNG)** | Direct Google | $20 | ~$18 (arbitrage) | $2 |
| **Total** | | **$250** | **$243** | **$7** |

**Verdict:** Minimal cost savings (~3%). OpenRouter's value is **resilience** and **model diversity**, not cost reduction.

---

## Recommendation

### For PAI

**Phase 1 (Immediate):**  
Add OpenRouter as a **Tier 2 fallback** in Inference.ts. Keep Ollama as primary, direct APIs as Tier 3. No behavior change, just adds resilience.

**Phase 2 (Optional):**  
Enable OpenRouter for **non-interactive services** (Functions API, Local Council) to test reliability. Monitor latency and cost.

**Phase 3 (User choice):**  
Add `--provider openrouter` flag to OpenCode. Let user opt-in for specific workflows.

### For OpenCode Users

If you want 400+ models without per-provider setup:
1. Sign up for OpenRouter
2. Configure OpenCode with OpenRouter baseURL
3. Enjoy automatic failover and cost arbitrage

If you want lowest latency and full control:
1. Keep direct provider APIs
2. Use Ollama for free local inference
3. Manually handle failover (switch providers if one is down)

---

## Conclusion

**OpenRouter and OpenCode are complementary, not competing:**

- **OpenRouter** = LLM infrastructure (routing, failover, billing)
- **OpenCode** = Developer productivity (coding, planning, git)

**PAI already has mini-OpenRouter logic** (Inference.ts + matrix routing). Adding actual OpenRouter as a backend would:

✅ Add 400+ models instantly  
✅ Enable automatic provider failover  
✅ Simplify API key management  
✅ Support data residency compliance  

⚠️ Add external dependency (OpenRouter uptime)  
⚠️ Add ~50-100ms latency overhead  
⚠️ Reduce control over provider selection  

**Best use case:** Background services (Functions API, Local Council) where latency tolerance is higher and availability is critical.

**Avoid for:** Primary interactive engines (PNC, PNK) where latency matters and direct provider control is preferred.

---

**Related Docs:**
- `PAI/Tools/Inference.ts` — Current model routing
- `PAI/Tools/inference-matrix.json` — Task-based routing config
- `PAI/DOCUMENTATION/MODEL_BENCHMARK_SYSTEM.md` — Quality calibration design
- `PAI/DOCUMENTATION/ENGINE_DAEMON_ARCHITECTURE.md` — Daemon vs interactive services
