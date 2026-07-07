# OpenRouter Integration Guide for PAI

**Quick-start guide for adding OpenRouter as a provider backend**

---

## Quick Setup (5 minutes)

### 1. Get OpenRouter API Key

```bash
# Visit https://openrouter.ai/settings/keys
# Create account, add credits, generate API key

# Add to PAI secrets
echo "OPENROUTER_API_KEY=sk-or-v1-..." >> ~/.config/PAI/secrets.env
```

### 2. Test OpenRouter Access

```bash
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -d '{
    "model": "anthropic/claude-sonnet-4-6",
    "messages": [
      {"role": "user", "content": "Say hello!"}
    ]
  }'
```

Expected response:
```json
{
  "id": "gen-...",
  "model": "anthropic/claude-sonnet-4-6",
  "choices": [{
    "message": {
      "role": "assistant",
      "content": "Hello! How can I help you today?"
    }
  }],
  "usage": {
    "prompt_tokens": 8,
    "completion_tokens": 9,
    "total_tokens": 17
  }
}
```

---

## Phase 1: Add OpenRouter to Inference.ts

### Add OpenRouter Inference Path

```typescript
// PAI/Tools/Inference.ts

/**
 * OpenRouter API Path
 */
async function inferenceOpenRouter(
  options: InferenceOptions, 
  modelName: string
): Promise<InferenceResult> {
  const startTime = Date.now();
  const apiKey = process.env.OPENROUTER_API_KEY;
  
  if (!apiKey) {
    return { 
      success: false, 
      output: '', 
      error: 'OPENROUTER_API_KEY missing', 
      latencyMs: 0, 
      model: modelName 
    };
  }

  try {
    const messages = [];
    if (options.systemPrompt) {
      messages.push({ role: 'system', content: options.systemPrompt });
    }
    messages.push({ role: 'user', content: options.userPrompt });

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
        messages,
        temperature: options.options?.temperature,
        max_tokens: options.options?.max_tokens || 4000,
      }),
      signal: options.timeout ? AbortSignal.timeout(options.timeout) : undefined,
    });

    if (!response.ok) {
      const error = await response.text();
      return { 
        success: false, 
        output: '', 
        error: `OpenRouter API error: ${response.status} ${error}`, 
        latencyMs: Date.now() - startTime, 
        model: modelName 
      };
    }

    const result = await response.json();
    const output = result.choices[0]?.message?.content || '';
    const latencyMs = Date.now() - startTime;

    // Record cost
    if (options.sessionId && result.usage) {
      recordSessionCost(
        options.sessionId, 
        modelName, 
        result.usage.prompt_tokens, 
        result.usage.completion_tokens
      );
    }

    recordModelPerf(modelName, latencyMs, true, options.userPrompt.length);

    let parsed = undefined;
    if (options.expectJson) {
      try {
        parsed = JSON.parse(output);
      } catch (e) {
        console.warn(`[Inference] JSON parse failed for ${modelName}: ${e}`);
      }
    }

    return { success: true, output, parsed, latencyMs, model: modelName };
  } catch (error) {
    const latencyMs = Date.now() - startTime;
    recordModelPerf(modelName, latencyMs, false, options.userPrompt.length);
    return { 
      success: false, 
      output: '', 
      error: String(error), 
      latencyMs, 
      model: modelName 
    };
  }
}

// Add to routing logic in runInference()
export async function runInference(options: InferenceOptions): Promise<InferenceResult> {
  // ... existing code ...

  // Detect OpenRouter models
  if (model.startsWith('openrouter/')) {
    const orModel = model.replace('openrouter/', '');
    return await inferenceOpenRouter(options, orModel);
  }

  // ... rest of existing routing ...
}
```

### Update inference-matrix.json

```json
{
  "routing_matrix": {
    "fast_classification": {
      "primary": "ollama/qwen3:8b",
      "fallback": "ollama/gemma4:e2b",
      "openrouter_fallback": "openrouter/anthropic/claude-haiku-4",
      "tier": "LOCAL"
    },
    "complex_reasoning": {
      "primary": "ollama/gemma2:9b",
      "fallback": "claude/opus",
      "openrouter_fallback": "openrouter/anthropic/claude-opus-4",
      "tier": "LOCAL"
    },
    "multimodal": {
      "primary": "gemini-1.5-flash",
      "fallback": "openrouter/google/gemini-1.5-pro",
      "tier": "CLOUD-T1"
    }
  },
  "openrouter_enabled": false,
  "openrouter_failover": true
}
```

### Test

```bash
# Test OpenRouter routing
bun ~/.claude/PAI/Tools/Inference.ts \
  --task complex_reasoning \
  --force-model openrouter/anthropic/claude-sonnet-4-6 \
  --prompt "What is 2+2?"

# Should return response via OpenRouter
```

---

## Phase 2: Enable for Background Services

### Functions API

```typescript
// PAI/Tools/FunctionsAPI/server.ts

import { Anthropic } from "@anthropic-ai/sdk";

const USE_OPENROUTER = process.env.OPENROUTER_ENABLED === 'true';

const anthropic = new Anthropic({
  apiKey: USE_OPENROUTER 
    ? process.env.OPENROUTER_API_KEY 
    : process.env.ANTHROPIC_API_KEY,
  baseURL: USE_OPENROUTER 
    ? "https://openrouter.ai/api/v1" 
    : undefined,
  defaultHeaders: USE_OPENROUTER ? {
    "HTTP-Referer": "https://pai.duane.ai",
    "X-Title": "PAI Functions API",
  } : undefined,
});

// All existing code unchanged — SDK handles routing
```

```bash
# Enable OpenRouter for Functions API
echo "OPENROUTER_ENABLED=true" >> ~/.config/PAI/secrets.env

# Restart service
systemctl --user restart pai-functions.service

# Test
curl http://localhost:8890/v1/inference \
  -H "Content-Type: application/json" \
  -d '{"prompt":"Hello!","model":"claude-sonnet-4-6"}'
```

### Local Council

```typescript
// PAI/Tools/LocalCouncil/server.ts

// Same pattern — add USE_OPENROUTER flag
const USE_OPENROUTER = process.env.OPENROUTER_ENABLED === 'true';

const clients = {
  anthropic: new Anthropic({
    apiKey: USE_OPENROUTER ? process.env.OPENROUTER_API_KEY : process.env.ANTHROPIC_API_KEY,
    baseURL: USE_OPENROUTER ? "https://openrouter.ai/api/v1" : undefined,
    defaultHeaders: USE_OPENROUTER ? {
      "HTTP-Referer": "https://pai.duane.ai",
      "X-Title": "PAI Local Council",
    } : undefined,
  }),
  // ... other clients unchanged
};
```

---

## Phase 3: Add to OpenCode (Optional)

### OpenCode Config

```jsonc
// ~/.config/opencode/opencode.json

{
  "providers": [
    {
      "id": "anthropic",
      "name": "Anthropic (Direct)",
      "baseUrl": "https://api.anthropic.com/v1",
      "apiKeyEnv": "ANTHROPIC_API_KEY",
      "models": ["claude-sonnet-4-6", "claude-opus-4"]
    },
    {
      "id": "openrouter",
      "name": "OpenRouter (400+ Models)",
      "baseUrl": "https://openrouter.ai/api/v1",
      "apiKeyEnv": "OPENROUTER_API_KEY",
      "headers": {
        "HTTP-Referer": "https://pai.duane.ai",
        "X-Title": "PAI OpenCode"
      },
      "models": [
        "anthropic/claude-sonnet-4-6",
        "openai/gpt-4o",
        "google/gemini-1.5-pro",
        "meta-llama/llama-3.3-70b-instruct",
        "mistralai/mistral-large-2",
        "deepseek/deepseek-chat",
        "qwen/qwen-2.5-72b-instruct"
      ]
    }
  ],
  "defaultProvider": "anthropic",  // Change to "openrouter" to enable
  "defaultModel": "claude-sonnet-4-6"
}
```

### CLI Flag

```bash
# Use direct Anthropic (default)
opencode

# Use OpenRouter backend
opencode --provider openrouter

# Use OpenRouter with specific model
opencode --provider openrouter --model meta-llama/llama-3.3-70b-instruct
```

---

## Monitoring & Observability

### Track OpenRouter Usage

```bash
# Add to PAI/Tools/Cost.ts

export function recordOpenRouterCost(
  sessionId: string,
  model: string,
  inputTokens: number,
  outputTokens: number
): void {
  const cost = calculateOpenRouterCost(model, inputTokens, outputTokens);
  
  appendFileSync(
    join(PAI_DIR, "MEMORY/STATE.claude/cost-log.jsonl"),
    JSON.stringify({
      ts: new Date().toISOString(),
      session_id: sessionId,
      provider: "openrouter",
      model,
      input_tokens: inputTokens,
      output_tokens: outputTokens,
      cost_usd: cost,
    }) + "\n"
  );
}

function calculateOpenRouterCost(
  model: string, 
  inputTokens: number, 
  outputTokens: number
): number {
  // OpenRouter pricing per 1M tokens
  const pricing: Record<string, { input: number; output: number }> = {
    "anthropic/claude-sonnet-4-6": { input: 3.0, output: 15.0 },
    "openai/gpt-4o": { input: 2.5, output: 10.0 },
    "google/gemini-1.5-pro": { input: 1.25, output: 5.0 },
    "meta-llama/llama-3.3-70b-instruct": { input: 0.9, output: 0.9 },
    // ... add more as needed
  };

  const rates = pricing[model] || { input: 0, output: 0 };
  return (inputTokens * rates.input + outputTokens * rates.output) / 1_000_000;
}
```

### Health Check Dashboard

```typescript
// Add to PAI Command Center (pai-command-center.service)

async function getOpenRouterHealth() {
  try {
    const response = await fetch('https://status.openrouter.ai/api/v2/summary.json');
    const status = await response.json();
    return {
      status: status.status.indicator === 'none' ? 'healthy' : 'degraded',
      updated: status.page.updated_at,
    };
  } catch {
    return { status: 'unknown', updated: null };
  }
}

// Display in Command Center UI
app.get('/api/health', async (req, res) => {
  const health = {
    ollama: await getOllamaHealth(),
    anthropic: await getAnthropicHealth(),
    openrouter: await getOpenRouterHealth(),  // NEW
  };
  res.json(health);
});
```

---

## Failover Strategy

### Automatic Failover Logic

```typescript
// PAI/Tools/Inference.ts

async function inferenceWithFailover(options: InferenceOptions): Promise<InferenceResult> {
  const taskConfig = MATRIX.routing_matrix[options.taskType || 'default'];
  
  // Tier 1: Try primary (usually Ollama)
  if (taskConfig.primary.startsWith('ollama/')) {
    const result = await inferenceOllama(options, taskConfig.primary);
    if (result.success) return result;
  }

  // Tier 2: Try OpenRouter fallback (if enabled)
  if (MATRIX.openrouter_failover && taskConfig.openrouter_fallback) {
    console.log(`[Inference] Primary failed, trying OpenRouter: ${taskConfig.openrouter_fallback}`);
    const result = await inferenceOpenRouter(options, taskConfig.openrouter_fallback);
    if (result.success) return result;
  }

  // Tier 3: Try direct API fallback
  const fallbackModel = taskConfig.fallback;
  console.log(`[Inference] OpenRouter failed, trying direct: ${fallbackModel}`);
  
  if (fallbackModel.startsWith('claude/')) {
    return await inferenceClaudeSDK(options, fallbackModel);
  } else if (fallbackModel.startsWith('gemini/')) {
    return await inferenceGeminiSDK(options, fallbackModel);
  } else if (fallbackModel.startsWith('openai/')) {
    return await inferenceOpenAI(options, fallbackModel);
  }

  return { 
    success: false, 
    output: '', 
    error: 'All providers failed', 
    latencyMs: 0, 
    model: 'none' 
  };
}
```

### Enable Failover

```json
// PAI/Tools/inference-matrix.json

{
  "openrouter_failover": true,  // Enable OpenRouter as Tier 2
  "routing_matrix": {
    "complex_reasoning": {
      "primary": "ollama/gemma2:9b",              // Tier 1 (free)
      "openrouter_fallback": "openrouter/anthropic/claude-opus-4",  // Tier 2 (resilient)
      "fallback": "claude/opus"                   // Tier 3 (direct)
    }
  }
}
```

---

## Cost Comparison

### Monthly Estimate (Duane's Usage)

| Task Type | Volume | Current Cost | Via OpenRouter | Difference |
|-----------|--------|--------------|----------------|------------|
| **Fast classification** | 10K calls | $0 (Ollama) | $0 (Ollama) | $0 |
| **Summarization** | 5K calls | $0 (Ollama) | $0 (Ollama) | $0 |
| **Complex reasoning** | 2K calls | $60 (Claude Opus) | ~$58 (OR arbitrage) | -$2 |
| **Multimodal** | 500 calls | $10 (Gemini) | ~$9 (OR) | -$1 |
| **Code generation** | 3K calls | $0 (Ollama) | $0 (Ollama) | $0 |
| **Creative writing** | 1K calls | $30 (Claude) | ~$30 (OR) | $0 |
| **Total** | | **$100** | **$97** | **-$3** |

**Verdict:** ~3% savings. Primary benefit is **resilience** and **model diversity**, not cost.

---

## Security Considerations

### Data Privacy

OpenRouter's default data policy:
- **Logs prompts** for debugging (retained 30 days)
- **Shares usage stats** with providers
- **No training** on your data (per their ToS)

To disable logging:

```typescript
const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
  headers: {
    'Authorization': `Bearer ${apiKey}`,
    'X-OpenRouter-Config': JSON.stringify({
      provider: {
        allow_fallbacks: true,
        data_collection: "deny",  // Disable logging
      }
    })
  },
  // ... rest of request
});
```

### API Key Rotation

```bash
# Add OpenRouter to KeyRotation skill
# ~/.claude/skills/KeyRotation/References/RedeployMap.md

## OpenRouter API Key

**Stores:**
- `~/.config/PAI/secrets.env` (OPENROUTER_API_KEY)

**Consumers:**
- pai-functions.service (if OPENROUTER_ENABLED=true)
- pai-local-council.service (if OPENROUTER_ENABLED=true)
- OpenCode (if defaultProvider=openrouter)

**Rotation steps:**
1. Generate new key at https://openrouter.ai/settings/keys
2. Update `~/.config/PAI/secrets.env`
3. Restart: `systemctl --user restart pai-functions.service pai-local-council.service`
4. Revoke old key in OpenRouter dashboard
```

---

## Testing Checklist

### Smoke Tests

```bash
# 1. Direct inference test
bun ~/.claude/PAI/Tools/Inference.ts \
  --force-model openrouter/anthropic/claude-sonnet-4-6 \
  --prompt "Count to 5"

# 2. Functions API test
curl http://localhost:8890/v1/inference \
  -H "Content-Type: application/json" \
  -d '{"prompt":"Say hello","model":"claude-sonnet-4-6"}'

# 3. Failover test (stop Ollama, trigger fallback)
ssh pai-primary "systemctl stop ollama.service"
bun ~/.claude/PAI/Tools/Inference.ts \
  --task fast_classification \
  --prompt "Classify: positive"
# Should fallback to OpenRouter

# 4. Cost tracking test
grep openrouter ~/MEMORY/STATE.claude/cost-log.jsonl | tail -5

# 5. OpenCode test
opencode --provider openrouter --model anthropic/claude-sonnet-4-6
# Chat: "Create a hello.py file"
```

### Performance Tests

```bash
# Latency comparison
bun ~/.claude/PAI/Tools/ModelMatrix/benchmark.ts \
  --models "claude/sonnet,openrouter/anthropic/claude-sonnet-4-6" \
  --task "complex_reasoning" \
  --trials 10

# Expected: OpenRouter adds ~50-100ms overhead
```

---

## Rollback Plan

### If OpenRouter Integration Causes Issues

```bash
# 1. Disable for background services
sed -i 's/OPENROUTER_ENABLED=true/OPENROUTER_ENABLED=false/' ~/.config/PAI/secrets.env
systemctl --user restart pai-functions.service pai-local-council.service

# 2. Disable failover in matrix
# Edit PAI/Tools/inference-matrix.json:
{
  "openrouter_failover": false
}

# 3. Remove OpenRouter from OpenCode
# Edit ~/.config/opencode/opencode.json:
{
  "defaultProvider": "anthropic"  # Change from "openrouter"
}

# 4. Verify direct APIs work
bun ~/.claude/PAI/Tools/Inference.ts \
  --force-model claude/sonnet \
  --prompt "Test"
```

---

## Next Steps

1. **Week 1:** Add OpenRouter to Inference.ts, test routing
2. **Week 2:** Enable for Functions API (low-risk background service)
3. **Week 3:** Monitor latency, cost, errors for 1 week
4. **Week 4:** If stable, enable for Local Council
5. **Month 2:** Optionally add to OpenCode as user choice

---

**Related Docs:**
- `PAI/DOCUMENTATION/OPENROUTER_VS_OPENCODE.md` — Full comparison
- `PAI/Tools/Inference.ts` — Inference dispatcher
- `PAI/Tools/inference-matrix.json` — Routing config
- `PAI/DOCUMENTATION/MODEL_BENCHMARK_SYSTEM.md` — Quality calibration
