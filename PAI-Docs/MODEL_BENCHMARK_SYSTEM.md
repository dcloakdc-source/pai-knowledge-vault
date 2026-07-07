# Model Benchmark & Calibration System

**Status:** Design Specification | **Version:** v1.0 | **Date:** 2026-05-14

## Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [Current State Audit](#2-current-state-audit)
3. [Architecture](#3-architecture)
4. [Schema Definitions](#4-schema-definitions)
5. [Test Suite Design](#5-test-suite-design)
6. [Execution Pipeline](#6-execution-pipeline)
7. [Grading System](#7-grading-system)
8. [Results Integration](#8-results-integration)
9. [Implementation Phases](#9-implementation-phases)
10. [Operations Runbook](#10-operations-runbook)
11. [File Map](#11-file-map)

---

## 1. Problem Statement

PAI's model routing (`inference-matrix.json`) maps 13 task types to primary/fallback models. These mappings were **editorially assigned** based on reputation and rough capability tags — not empirically calibrated. The existing `benchmarks.jsonl` tracks only speed (latency, throughput), not output quality.

**What we can't answer today:**
- Is `ollama/gemma2:9b` actually better at summarization than `ollama/gemma3:4b`?
- Does `ollama/deepseek-r1:7b` produce reasoning at parity with `claude/sonnet`?
- When does `ollama/qwen3:8b` cross the quality threshold to replace `claude/haiku`?
- Is `inception/mercury-2` with `low` reasoning effort "good enough" for classification?

**What the system must deliver:**
1. **Per-task-type quality scores** for every model (local + cloud)
2. **Ranked model recommendations** updated when new models arrive or quality shifts
3. **Cost-quality tradeoff analysis** (when does free beat paid?)
4. **Drift detection** — when a model update silently degrades a capability

---

## 2. Current State Audit

### 2.1 What Exists (Building Blocks)

| Component | File | Role | Coverage |
|-----------|------|------|----------|
| **ModelMatrix catalog** | `PAI/Tools/ModelMatrix/index.ts` | 20 models cataloged with capability tags, cost, context windows | Full: all active models |
| **Speed benchmarks** | `PAI/Tools/ModelMatrix/benchmarks.jsonl` | 30 rows of latency/throughput for Ollama models | Speed only |
| **Inference dispatcher** | `PAI/Tools/Inference.ts` | Routes 11 task types to models, records perf to `shared-brain/model-performance.jsonl` | Full: all providers |
| **Routing matrix** | `PAI/Tools/inference-matrix.json` | 11 task → primary/fallback mappings | Static/editorial |
| **Local model discovery** | `ModelMatrix/index.ts discover` | Auto-discovers 22 Ollama models, writes `local-models.json` | Full: Ollama |
| **Evals: CompareModels** | `opencode/skills/Utilities/Evals/Workflows/CompareModels.md` | Multi-model comparison with rubric grading, p-values, cost/latency tracking | Workflow defined, no task-specific suites |
| **Evals: TrialRunner** | `opencode/skills/Utilities/Evals/Tools/TrialRunner.ts` | Multi-trial execution with pass@k / pass^k, 6 grader types | Framework ready |
| **Engine pricing** | `PAI/Tools/ModelMatrix/engine-pricing.json` | All provider costs per 1M tokens (Anthropic/Google/OpenAI/Ollama/OpenCode) | Full |
| **SettingsLab** | `skills/SettingsLab/` | `harvest → compare → refine → retest → apply` loop, Ollama A/B replays, rubric scoring | Infrastructure ready |

### 2.2 Gap Analysis

| Gap | Severity | What's Missing |
|-----|----------|---------------|
| **No quality benchmarks** | Critical | Only speed data exists. Quality is entirely unmeasured for local models. |
| **No task-specific test suites** | Critical | CompareModels workflow exists but no test cases for classification, summarization, coding, etc. |
| **Static routing matrix** | High | `inference-matrix.json` is hand-authored, never empirically updated. |
| **No cross-model quality comparison** | High | Can't compare `ollama/gemma2:9b` to `claude/haiku` on the same task. |
| **No drift detection** | Medium | When Ollama pulls an updated model, quality changes invisibly. |
| **Benchmark for Ollama only** | Medium | `runBenchmark()` in index.ts only targets Ollama. |
| **No automated pipeline** | Medium | Benchmarks run ad-hoc; no scheduled recalibration. |

---

## 3. Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         MODEL BENCHMARK SYSTEM                              │
│                                                                             │
│  ┌──────────┐   ┌──────────────┐   ┌──────────────┐   ┌─────────────────┐  │
│  │ Test     │   │ Inference    │   │ Grading      │   │ Results         │  │
│  │ Suites   │──▶│ Engine       │──▶│ Engine       │──▶│ Aggregator      │  │
│  │          │   │              │   │              │   │                 │  │
│  │ Per task │   │ Inference.ts │   │ Evals graders│   │ Per-model       │  │
│  │ type:    │   │ (Ollama/     │   │ (rubric/     │   │ scores per      │  │
│  │ - inputs │   │  Claude/     │   │  assertion/  │   │ task type       │  │
│  │ - rubric │   │  Gemini/     │   │  code-based) │   │                 │  │
│  │ - anti-  │   │  OpenAI)     │   │              │   │                 │  │
│  │   criteria   │              │   │              │   │                 │  │
│  └──────────┘   └──────────────┘   └──────────────┘   └────────┬────────┘  │
│                                                                 │          │
│  ┌──────────────────────────────────────────────────────────────┘          │
│  │                                                                         │
│  ▼                                                                         │
│  ┌──────────────┐   ┌───────────────┐   ┌────────────────┐                 │
│  │ Matrix       │   │ Drift         │   │ Operations     │                 │
│  │ Updater      │   │ Detector      │   │ Dashboard      │                 │
│  │              │   │               │   │                │                 │
│  │ Updates      │   │ Compares      │   │ PAI Command    │                 │
│  │ inference-   │   │ current vs    │   │ Center:        │                 │
│  │ matrix.json  │   │ prior scores  │   │ 192.168.50.20  │                 │
│  │              │   │ → alerts      │   │ :8766/Models   │                 │
│  └──────────────┘   └───────────────┘   └────────────────┘                 │
│                                                                             │
│  SCHEDULE:                                                                  │
│    Daily 3am  → discover new Ollama models + speed benchmark                │
│    Weekly Sun 4am → full quality benchmark (all task types, all models)     │
│    On push  → partial benchmark (new/changed models only)                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Schema Definitions

### 4.1 Quality Benchmark Record (`quality-benchmarks.jsonl`)

Extends the existing speed-only benchmark with quality dimensions:

```jsonl
{
  "ts": "2026-05-14T04:00:00.000Z",
  "run_id": "qb-20260514-001",
  "task_type": "summarization",
  "model": "ollama/gemma2:9b",

  "speed": {
    "latency_ms": 4200,
    "throughput_tps": 45,
    "input_tokens": 380,
    "output_tokens": 120
  },

  "quality": {
    "accuracy": 0.87,
    "completeness": 0.91,
    "format_compliance": 1.0,
    "anti_criterion_pass": true,
    "composite_score": 0.88,
    "judge_model": "claude/sonnet",
    "judge_confidence": 0.85
  },

  "cost": {
    "input_cost_usd": 0.0,
    "output_cost_usd": 0.0,
    "total_cost_usd": 0.0,
    "cost_source": "engine-pricing.json"
  },

  "verdict": "pass",
  "test_case_id": "summ-003"
}
```

### 4.2 Aggregated Model Score (per task type)

```typescript
interface ModelTaskScore {
  model_id: string;
  task_type: TaskType;
  
  // Quality (0.0-1.0)
  composite_quality: number;
  quality_breakdown: Record<string, number>;  // e.g. { accuracy: 0.87, completeness: 0.91 }
  
  // Speed
  avg_latency_ms: number;
  avg_throughput_tps: number;
  p95_latency_ms: number;
  
  // Cost
  cost_per_1k_output: number;  // USD
  cost_per_1k_input: number;
  
  // Reliability
  pass_rate: number;           // % of test cases where verdict=pass
  trial_count: number;
  std_dev: number;             // quality variance across trials
  
  // Comparison
  rank: number;                // 1 = best for this task type
  efficiency_score: number;    // composite_quality / (cost_per_1k_output + epsilon)
  
  // Metadata
  benchmarked_at: string;
  prior_score?: number;        // for drift detection
  quality_delta?: number;      // change from prior benchmark
}
```

### 4.3 Calibrated Routing Entry

Extends the current `TASK_ROUTING` with quality evidence:

```typescript
interface CalibratedRouting {
  task_type: TaskType;
  primary: string;     // model ID
  fallback: string;
  
  // Evidence
  primary_quality: number;
  fallback_quality: number;
  
  // Cost-quality frontier
  best_quality: string;       // model with highest quality (may be cloud)
  best_free: string;          // best free (local) model
  best_cost_efficiency: string; // best quality-per-dollar
  
  // Metadata
  benchmark_run_id: string;
  updated_at: string;
  confidence: number;         // 0-1, based on trial count and variance
}
```

---

## 5. Test Suite Design

### 5.1 Suite Structure

Each task type gets a directory at:
```
PAI/Tools/ModelMatrix/test-suites/{task_type}/
├── config.yaml          # Suite name, task type, grader config
├── inputs/              # Test prompts
│   ├── 001.txt
│   ├── 002.txt
│   └── ...
├── rubric.md            # LLM-judge grading rubric
├── anti-criteria.md     # What models must NOT produce
├── reference/           # Ground truth / reference answers (for exact-match tasks)
│   ├── 001.json
│   └── 002.json
└── grader.ts            # Optional custom TypeScript grader
```

### 5.2 Suite Design Per Task Type

#### `fast_classification` — 20 test cases

**Input:** 20 text snippets (tweets, comments, reviews, CVEs)
**Task:** Classify as one of 3-5 labels (sentiment: positive/negative/neutral; CVE severity: critical/high/medium/low)
**Grader:** `string_match` + `regex_match` against ground truth
**Anti-criterion:** Must not output more than 3 sentences of explanation. Just the label.
**Models to test:** `ollama/llama3.2:3b`, `ollama/nemotron-mini`, `ollama/qwen3:1.7b`, `ollama/qwen3:4b`, `claude/haiku`, `gemini-1.5-flash`, `inception/mercury-2` (low effort)

**Example:**
```yaml
# inputs/001.json
{
  "id": "class-001",
  "prompt": "Classify this text as positive, negative, or neutral — reply with just one word:\n\n'I loved the new update, it's so much faster now!'",
  "expected": "positive",
  "anti_patterns": ["explanation", "because", "the sentiment is"]
}
```

#### `structured_json` — 15 test cases

**Input:** 15 unstructured text blocks (resumes, CVEs, meeting notes, product descriptions)
**Task:** Extract structured fields into JSON
**Grader:** JSON schema validation + field-level `string_match` against ground truth
**Anti-criterion:** Must not hallucinate fields not present in the source. Must not wrap in markdown code fences.
**Models to test:** `ollama/qwen2.5:7b`, `ollama/gemma2:9b`, `ollama/qwen3:8b`, `claude/haiku`, `claude/sonnet`, `gemini-1.5-flash`, `inception/mercury-2` (medium effort)

**Example:**
```yaml
# inputs/001.json
{
  "id": "json-001",
  "prompt": "Extract the following fields from this CVE description as JSON. Return ONLY the JSON object, no other text.\n\nCVE-2024-1234: A buffer overflow in the authentication module of Apache Kafka 3.5.0 allows remote attackers to execute arbitrary code via a specially crafted JWT token. Fixed in version 3.5.1.",
  "expected": {
    "cve_id": "CVE-2024-1234",
    "vulnerability_type": "buffer overflow",
    "affected_component": "authentication module",
    "affected_product": "Apache Kafka",
    "affected_version": "3.5.0",
    "fixed_version": "3.5.1",
    "attack_vector": "remote",
    "requires_auth": false
  },
  "anti_fields": ["severity", "cvss_score", "published_date"]  # Must not hallucinate these
}
```

#### `summarization` — 10 test cases

**Input:** 10 articles (800-3000 words each) from tech, security, policy
**Task:** Summarize in ≤4 sentences, capturing thesis, key evidence, conclusion
**Grader:** `llm_rubric` (accuracy, completeness, conciseness) scored by `claude/sonnet`
**Anti-criterion:** Must not introduce facts not in the source. Must not editorialize.
**Models to test:** `ollama/gemma2:9b`, `ollama/deepseek-r1:7b`, `ollama/gemma3:4b`, `ollama/qwen3:8b`, `ollama/tulu3`, `claude/haiku`, `claude/sonnet`, `gemini-1.5-flash`, `gemini-1.5-pro`

**Rubric (rubric.md):**
```
Score each summary on three dimensions (1-5):

1. ACCURACY: Does the summary contain only facts from the source?
   5 = All claims directly sourced from the article
   1 = Contains fabricated or incorrect claims

2. COMPLETENESS: Does it capture the main thesis, key evidence, and conclusion?
   5 = All three elements present and proportionally weighted
   1 = Misses the main thesis entirely

3. CONCISENESS: Is it 4 sentences or fewer, with no wasted words?
   5 = Exactly 4 tight sentences, zero filler
   1 = Exceeds 10 sentences or contains significant redundancy
```

#### `multi_step_reasoning` — 10 test cases

**Input:** 10 logic/math/planning problems requiring step-by-step reasoning
**Task:** Solve the problem, showing reasoning steps, then give final answer on a line by itself
**Grader:** `regex_match` on final answer line + `llm_rubric` for reasoning quality
**Anti-criterion:** Must not give the answer without showing work. Must not contradict itself across steps.
**Models to test:** `ollama/deepseek-r1:7b`, `ollama/gemma2:9b`, `ollama/qwen3:8b`, `ollama/command-r7b`, `claude/sonnet`, `gemini-1.5-pro`, `inception/mercury-2` (high effort)

**Example:**
```yaml
# inputs/001.json
{
  "id": "reason-001",
  "prompt": "A bat and ball cost $1.10 in total. The bat costs $1.00 more than the ball. How much does the ball cost? Show your reasoning step by step, then give the final answer on a line that starts with 'ANSWER:'.",
  "expected_answer": "0.05",
  "expected_answer_line": "ANSWER: 0.05"
}
```

#### `code_generation` — 10 test cases

**Input:** 10 programming tasks (implement a function, fix a bug, write a query)
**Task:** Produce working code
**Grader:** `binary_tests` (run against test file) + `static_analysis` (Biome lint) + `tool_calls` (must not call external APIs)
**Anti-criterion:** Must not produce code that doesn't compile or has obvious logic errors.
**Models to test:** `claude/sonnet`, `claude/haiku`, `gemini-1.5-pro`, `ollama/qwen2.5:7b`, `ollama/deepseek-r1:7b`, `ollama/qwen3:8b`, `inception/mercury-2` (high effort)

**Example:**
```yaml
# inputs/001.json
{
  "id": "code-001",
  "prompt": "Write a TypeScript function `parseCVEList(text: string): CVE[]` that parses a text block containing multiple CVE IDs (format: CVE-YYYY-NNNNN) and returns them as an array of objects with `id`, `year`, and `number` fields. Handle duplicates by returning each CVE only once.",
  "test_file": "tests/code-001.test.ts",
  "static_checks": ["biome lint", "tsc --noEmit"]
}
```

#### `complex_reasoning` — 5 test cases

**Input:** 5 architect-level decision problems with tradeoffs
**Task:** Analyze tradeoffs, recommend approach, justify with evidence
**Grader:** `llm_rubric` (depth, tradeoff awareness, evidence quality) + `natural_language_assert`
**Models to test:** `claude/sonnet`, `claude/opus`, `gemini-1.5-pro`, `gemini-2.5-pro`, `ollama/deepseek-r1:7b`, `ollama/granite4:small-h`

#### `long_doc_analysis` — 5 test cases

**Input:** 5 documents of 8K-15K tokens → answer 3 factual questions
**Task:** Answer questions using only information from the document
**Grader:** `string_match` against expected answers + `llm_rubric` for faithfulness
**Models to test:** `gemini-1.5-pro`, `gemini-1.5-flash`, `gemini-2.5-pro`, `claude/sonnet`, `claude/haiku`

#### `creative_writing` — 5 test cases

**Input:** 5 writing prompts (scene description, dialogue, technical explanation for lay audience)
**Task:** Write engaging prose matching the prompt
**Grader:** `pairwise_comparison` (position-swapped, scored by claude/sonnet)
**Models to test:** `claude/opus`, `claude/sonnet`, `gemini-1.5-pro`, `ollama/gemma2:9b`

#### `multimodal` — 5 test cases

**Input:** 5 image → description tasks
**Task:** Describe the image accurately
**Grader:** `llm_rubric` (accuracy, completeness, detail)
**Models to test:** `gemini-1.5-flash`, `gemini-1.5-pro`, `gemini-2.5-pro`

#### `default` — 10 test cases (general capability)

**Input:** Mixed prompts from all categories (3 classification, 3 JSON, 2 reasoning, 2 code)
**Task:** Respond appropriately per prompt type
**Grader:** Mixed (exact match for classification, rubric for others)
**Purpose:** Catch-all for unspecialized routing

### 5.3 Anti-Criteria Framework

Every test suite enforces anti-criteria — things the model must NOT do:

| Pattern | Check Method | Example |
|---------|-------------|---------|
| Hallucinated fields | Schema validation | Adding `cvss_score` to JSON output when source has none |
| Hedging language | Regex blacklist | "I think", "I'm not sure", "probably", "might be" |
| Refusal language | Regex blacklist | "I cannot", "I'm unable to", "As an AI" |
| Over-explanation | Token count | Classification task returning 500 words instead of 1 |
| Markdown wrapping | String match | Code fences in plain-text tasks |
| Unsourced claims | LLM assertion | Summary adding facts not in source article |
| Contradiction | LLM assertion | Reasoning chain where step 3 contradicts step 1 |

---

## 6. Execution Pipeline

### 6.1 Pipeline Architecture

```
┌──────────┐     ┌──────────┐     ┌───────────┐     ┌────────────┐     ┌───────────┐
│ Discover │────▶│  Run     │────▶│  Grade    │────▶│ Aggregate  │────▶│  Update   │
│ Models   │     │  Tests   │     │  Results  │     │ Scores     │     │  Matrix   │
└──────────┘     └──────────┘     └───────────┘     └────────────┘     └───────────┘
    │                 │                 │                 │                  │
    ▼                 ▼                 ▼                 ▼                  ▼
local-models.     quality-          per-case          per-model          inference-
json + speed      benchmarks.       quality           task scores        matrix.json
benchmarks.       jsonl             verdicts          + ranking          (updated)
jsonl
```

### 6.2 Pipeline Implementation

A single entry point: `PAI/Tools/ModelMatrix/calibrate.ts`

```bash
# Full calibration — all task types, all models (~15-30 min)
bun PAI/Tools/ModelMatrix/calibrate.ts --full

# Single task type — one suite, all models (~2-5 min)
bun PAI/Tools/ModelMatrix/calibrate.ts --task summarization

# Single model — one model, all task types (~5-10 min)
bun PAI/Tools/ModelMatrix/calibrate.ts --model ollama/gemma3:4b

# Drift check only — compare current vs prior scores, alert if delta > threshold
bun PAI/Tools/ModelMatrix/calibrate.ts --drift-check

# Dry run — show what would run without executing
bun PAI/Tools/ModelMatrix/calibrate.ts --dry-run
```

### 6.3 Pipeline Steps (Detailed)

**Step 1: Discover & Select Models**

```typescript
// 1a. Read catalog for all active models
const cloudModels = MODEL_CATALOG.filter(m => m.provider !== 'ollama');
const localModels = await discoverLocalModels();
const allModels = [...cloudModels, ...localModels];

// 1b. Filter to inference-capable (exclude embeddings)
const inferenceModels = allModels.filter(m => 
  m.type === 'llm' && !m.capabilities.includes('embedding')
);

// 1c. Speed benchmark any new models (single-shot for speed baseline)
for (const m of inferenceModels) {
  if (!hasRecentSpeedBenchmark(m.id)) {
    await runSpeedBenchmark(m.id);
  }
}
```

**Step 2: Run Test Suites**

```typescript
// For each task type × model, run all test cases
for (const suite of TEST_SUITES) {
  for (const model of inferenceModels) {
    // Skip if model doesn't support task (e.g., no multimodal for Ollama)
    if (!modelSupportsTask(model, suite.task_type)) continue;
    
    for (const testCase of suite.test_cases) {
      // Dispatch inference
      const result = await inference({
        systemPrompt: suite.system_prompt,
        userPrompt: testCase.prompt,
        taskType: suite.task_type,
        // Bypass matrix routing — pin to specific model
        options: { force_model: model.id }
      });
      
      // Store raw output for grading
      rawResults.push({ model: model.id, task_type: suite.task_type, 
                        test_case: testCase.id, output: result.output, 
                        latency_ms: result.latencyMs });
    }
  }
}
```

**Step 3: Grade Results**

```typescript
// Grade using the configured grader per suite
for (const raw of rawResults) {
  const suite = getSuite(raw.task_type);
  const testCase = suite.getTestCase(raw.test_case_id);
  
  const grade = await grade(suite.grader_config, {
    output: raw.output,
    expected: testCase.expected,
    anti_criteria: testCase.anti_patterns,
    rubric: suite.rubric
  });
  
  // Write quality benchmark record
  appendQualityBenchmark({
    ...raw,
    quality: grade,
    cost: calculateCost(raw.model, raw.output.length)
  });
}
```

**Step 4: Aggregate Scores**

```typescript
// Per model × task type
for (const [model, taskType] of modelTaskPairs) {
  const records = getQualityBenchmarks({ model, task_type: taskType });
  
  const score: ModelTaskScore = {
    model_id: model,
    task_type: taskType,
    composite_quality: mean(records.map(r => r.quality.composite_score)),
    quality_breakdown: aggregateBreakdown(records),
    avg_latency_ms: mean(records.map(r => r.speed.latency_ms)),
    pass_rate: records.filter(r => r.verdict === 'pass').length / records.length,
    trial_count: records.length,
    std_dev: stdDev(records.map(r => r.quality.composite_score)),
    rank: 0, // filled in step 5
    efficiency_score: 0,
    benchmarked_at: new Date().toISOString(),
    prior_score: prior?.composite_quality,
    quality_delta: prior ? score.composite_quality - prior.composite_quality : undefined
  };
  
  storeModelTaskScore(score);
}
```

**Step 5: Rank & Update Matrix**

```typescript
// For each task type, rank models by composite_quality descending
for (const taskType of TASK_TYPES) {
  const scores = getScoresForTask(taskType);
  scores.sort((a, b) => b.composite_quality - a.composite_quality);
  
  // Assign ranks, calculate efficiency
  for (let i = 0; i < scores.length; i++) {
    scores[i].rank = i + 1;
    scores[i].efficiency_score = scores[i].composite_quality / 
      (Math.max(scores[i].cost_per_1k_output, 0.0001));
  }
  
  // Determine routing
  const best = scores[0];
  const bestFree = scores.find(s => isFree(s.model_id));
  const fallback = scores[1] || best;
  
  updateRoutingMatrix(taskType, {
    primary: bestFree?.model_id || best.model_id,  // Prefer free if quality > threshold
    fallback: fallback.model_id,
    best_quality: best.model_id,
    best_free: bestFree?.model_id || null,
    best_cost_efficiency: scores.sort((a, b) => b.efficiency_score - a.efficiency_score)[0].model_id,
    benchmark_run_id: RUN_ID,
    confidence: calculateConfidence(scores)
  });
}
```

### 6.4 Scheduled Execution

| Trigger | Command | Scope |
|---------|---------|-------|
| Daily 3am | `bun ModelMatrix/index.ts discover` + `bun ModelMatrix/index.ts benchmark` | New models + speed |
| Weekly Sun 4am | `bun ModelMatrix/calibrate.ts --full` | Full quality calibration |
| On model pull/update | `bun ModelMatrix/calibrate.ts --model <new-model>` | Single model calibration |
| Manual | `bun ModelMatrix/calibrate.ts --task <task>` | Single task type |

---

## 7. Grading System

### 7.1 Grader Selection Per Task Type

| Task Type | Primary Grader | Secondary Grader | Judge Model |
|-----------|---------------|------------------|-------------|
| `fast_classification` | `string_match` (exact) | — | n/a (deterministic) |
| `structured_json` | `string_match` (schema + fields) | `regex_match` (format check) | n/a |
| `summarization` | `llm_rubric` (accuracy, completeness, conciseness) | `natural_language_assert` (no hallucination) | `claude/sonnet` |
| `multi_step_reasoning` | `regex_match` (answer line) | `llm_rubric` (reasoning quality) | `claude/sonnet` |
| `code_generation` | `binary_tests` (run test file) | `static_analysis` (lint + typecheck) | n/a |
| `complex_reasoning` | `llm_rubric` (depth, tradeoffs) | `natural_language_assert` | `claude/sonnet` |
| `long_doc_analysis` | `string_match` (answer accuracy) | `llm_rubric` (faithfulness) | `claude/haiku` |
| `creative_writing` | `pairwise_comparison` (position-swap) | — | `claude/sonnet` |
| `multimodal` | `llm_rubric` (accuracy, detail) | — | `claude/sonnet` |
| `default` | Mixed (per test case) | — | varies |

### 7.2 LLM Judge Configuration

```typescript
interface JudgeConfig {
  model: string;                    // Judge model (should NOT be the model being tested)
  temperature: number;              // 0 for deterministic grading
  position_swap_enabled: boolean;   // Swap order for pairwise comparisons
  ensemble_size: number;            // 1 = single judge, 3 = multi-judge consensus
  ensemble_models?: string[];       // For multi-judge: different models
  calibration_threshold: number;    // Minimum confidence to accept grade
  retry_on_low_confidence: boolean; // Re-grade if below threshold
}

const DEFAULT_JUDGE: JudgeConfig = {
  model: "claude/sonnet",
  temperature: 0,
  position_swap_enabled: true,
  ensemble_size: 1,
  calibration_threshold: 0.7,
  retry_on_low_confidence: true
};
```

### 7.3 Judge Bias Mitigation

**Problem:** Using Claude to judge Claude may inflate Claude's scores.

**Mitigations:**
1. **Position swap** for pairwise comparisons (order randomized)
2. **Cross-judge validation** — spot-check 10% of grades with a different judge model (e.g., Gemini grades Claude's output)
3. **Bias baseline** — run an empty prompt through the judge to measure baseline score
4. **Anti-criteria** are deterministic and judge-independent — they catch specific failure modes regardless of judge bias
5. **Multi-judge consensus** for critical suites (code_generation, complex_reasoning) — use 3 judges with majority vote

---

## 8. Results Integration

### 8.1 Matrix Update Logic

The routing matrix (`inference-matrix.json`) is updated based on benchmark results with these rules:

```typescript
function determineRouting(taskType: TaskType, scores: ModelTaskScore[]): CalibratedRouting {
  const sorted = scores.sort((a, b) => b.composite_quality - a.composite_quality);
  const best = sorted[0];
  const bestFree = sorted.find(s => isFreeModel(s.model_id));
  const secondBest = sorted[1];
  
  // Routing rules:
  // 1. If best free model has quality > 0.8, use it as primary
  // 2. If best free model is within 10% of best paid, prefer free
  // 3. If no free model is adequate, use best paid
  // 4. Fallback is always second-best in the same tier
  
  let primary, fallback;
  
  if (bestFree && bestFree.composite_quality >= 0.8) {
    primary = bestFree.model_id;
    fallback = best.model_id !== primary ? best.model_id : sorted.find(s => s.model_id !== primary)!.model_id;
  } else if (bestFree && (bestFree.composite_quality / best.composite_quality) >= 0.9) {
    primary = bestFree.model_id;
    fallback = best.model_id;
  } else {
    primary = best.model_id;
    fallback = bestFree?.model_id || secondBest.model_id;
  }
  
  return {
    task_type: taskType,
    primary,
    fallback,
    primary_quality: scores.find(s => s.model_id === primary)!.composite_quality,
    fallback_quality: scores.find(s => s.model_id === fallback)!.composite_quality,
    best_quality: best.model_id,
    best_free: bestFree?.model_id || null,
    best_cost_efficiency: sorted.sort((a, b) => b.efficiency_score - a.efficiency_score)[0].model_id,
    benchmark_run_id: RUN_ID,
    updated_at: new Date().toISOString(),
    confidence: calculateConfidence(sorted),
  };
}
```

### 8.2 Matrix Update Safety

- **Never downgrade without confirmation** — if a model drops >15% in quality, flag for manual review instead of auto-downgrading
- **Confidence gating** — updates require ≥5 trials and std_dev < 0.15
- **Changelog** — old matrix version saved to `inference-matrix-{date}.json` before any update
- **Rollback command** — `bun ModelMatrix/calibrate.ts --rollback` restores previous matrix

### 8.3 Command Center Integration

The Command Center at `192.168.50.20:8766` → **Models** tab shows:

```
┌─────────────────────────────────────────────────────────────────┐
│ MODEL BENCHMARK DASHBOARD                          May 14, 2026 │
├─────────────────────────────────────────────────────────────────┤
│ TASK: Summarization                                              │
│ ┌──────┬─────────┬──────────┬────────┬───────┬───────┬────────┐ │
│ │ Rank │ Model   │ Quality  │ Pass%  │ Lat   │ Cost  │ Status │ │
│ ├──────┼─────────┼──────────┼────────┼───────┼───────┼────────┤ │
│ │ 1    │ claude/ │ 0.92     │ 100%   │ 2.1s  │ $0.03 │ ⭐best │ │
│ │      │ sonnet  │          │        │       │       │        │ │
│ │ 2    │ ollama/ │ 0.87     │ 90%    │ 4.2s  │ FREE  │ ✅free │ │
│ │      │ gemma2  │          │        │       │       │        │ │
│ │ 3    │ ollama/ │ 0.81     │ 80%    │ 3.8s  │ FREE  │        │ │
│ │      │ qwen3:8b│          │        │       │       │        │ │
│ │ 4    │ ollama/ │ 0.72     │ 70%    │ 3.1s  │ FREE  │ ⚠️drift│ │
│ │      │ gemma3:4│          │        │       │       │        │ │
│ └──────┴─────────┴──────────┴────────┴───────┴───────┴────────┘ │
│                                                                  │
│ Drift alert: gemma3:4b dropped from 0.81 → 0.72 since last run   │
│ [Investigate]                                                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Implementation Phases

### Phase 1: Test Suite Scaffold (Week 1)

Create test suites for the **3 highest-value task types**:
1. `summarization` — broadest use, most models compete
2. `fast_classification` — most volume, biggest savings potential
3. `structured_json` — critical for CVSS/data extraction pipelines

**Deliverables:**
- `PAI/Tools/ModelMatrix/test-suites/{summarization,fast_classification,structured_json}/` with inputs, rubrics, anti-criteria
- Manual benchmark run for top 5 models per task type
- Initial quality comparison table

### Phase 2: Grading Pipeline (Week 2)

Build the quality grading engine:
- Extend `Inference.ts` to support `force_model` parameter (bypass matrix routing)
- Add `quality_benchmarks.jsonl` format
- Implement graders per task type
- Build aggregation logic

**Deliverables:**
- `bun PAI/Tools/ModelMatrix/calibrate.ts --task <task>` working end-to-end
- Quality benchmark data for Phase 1 suites

### Phase 3: Remaining Suites + Matrix Integration (Week 3)

- Create remaining 8 test suites
- Build matrix update logic with safety gates
- Add drift detection
- Wire into `inference-matrix.json`

**Deliverables:**
- `bun PAI/Tools/ModelMatrix/calibrate.ts --full` working
- First empirically-calibrated `inference-matrix.json`

### Phase 4: Automation + Dashboard (Week 4)

- Wire into existing cron timers (`pai-model-benchmark.timer`)
- Add Command Center dashboard panel
- Set up drift alerts (ntfy)
- Write operations runbook

**Deliverables:**
- Automated weekly calibration
- Dashboard showing per-task rankings
- Drift alerts on model degradation

### Phase 5: Continuous Improvement (Ongoing)

- Add new models as they appear (model discovery is already automated)
- Rebalance test suites based on real failure patterns
- Calibrate judge bias (cross-model validation)
- Tune routing thresholds (when does free beat paid?)

---

## 10. Operations Runbook

### 10.1 View Current State

```bash
# Show current routing matrix with quality evidence
bun PAI/Tools/ModelMatrix/index.ts matrix

# Show model stats including quality (once populated)
bun PAI/Tools/ModelMatrix/calibrate.ts --stats

# Show rankings for a specific task
bun PAI/Tools/ModelMatrix/calibrate.ts --rankings summarization
```

### 10.2 Run Benchmarks

```bash
# Full weekly calibration (all task types, all models)
bun PAI/Tools/ModelMatrix/calibrate.ts --full

# Single task type (quick check after model update)
bun PAI/Tools/ModelMatrix/calibrate.ts --task summarization

# Single model (test a newly added model)
bun PAI/Tools/ModelMatrix/calibrate.ts --model ollama/gemma4:9b

# Drift check only (compare current vs prior, no full run)
bun PAI/Tools/ModelMatrix/calibrate.ts --drift-check
```

### 10.3 Investigate Quality Issues

```bash
# Find degraded models
bun PAI/Tools/ModelMatrix/calibrate.ts --drift-check

# See detailed breakdown for a model
bun PAI/Tools/ModelMatrix/calibrate.ts --inspect ollama/gemma3:4b

# Compare two models directly
bun PAI/Tools/ModelMatrix/calibrate.ts --compare ollama/gemma2:9b ollama/qwen3:8b --task summarization
```

### 10.4 Manage Routing

```bash
# Update matrix from latest benchmarks (auto with safety gates)
bun PAI/Tools/ModelMatrix/calibrate.ts --update-matrix

# Rollback to previous matrix
bun PAI/Tools/ModelMatrix/calibrate.ts --rollback

# Force specific routing (manual override)
bun PAI/Tools/ModelMatrix/calibrate.ts --override summarization primary=claude/sonnet
```

### 10.5 Scheduled Tasks

```bash
# View timer status
systemctl --user status pai-model-discover.timer
systemctl --user status pai-model-benchmark.timer

# Trigger manually
systemctl --user start pai-model-discover.service
systemctl --user start pai-model-benchmark.service

# View logs
journalctl --user -u pai-model-benchmark.service -n 50
```

### 10.6 Troubleshooting

| Problem | Check |
|---------|-------|
| Benchmarks failing | Is Ollama running? `curl http://192.168.50.20:11434/api/tags` |
| Cloud model failures | Check API keys: `echo $GOOGLE_API_KEY`, `cat ~/.config/PAI/secrets.env` |
| Judge bias suspected | Run `--compare` with cross-judge: swap judge model to gemini |
| Scores seem wrong | Inspect raw output: `bun calibrate.ts --inspect <model> --raw` |
| Drift false positive | Check trial count: <5 trials = low confidence, increase trials |

---

## 11. File Map

### New Files

```
PAI/Tools/ModelMatrix/
├── calibrate.ts              # Main calibration CLI (new)
├── calibration/
│   ├── pipeline.ts           # Execute benchmark pipeline (new)
│   ├── grader.ts             # Quality grading engine (new)
│   ├── aggregator.ts         # Score aggregation + ranking (new)
│   ├── matrix-updater.ts     # Safe matrix update with safety gates (new)
│   └── drift-detector.ts     # Compare current vs prior scores (new)
├── test-suites/              # Per-task-type test suites (new)
│   ├── fast_classification/
│   │   ├── config.yaml
│   │   ├── inputs/           # 20 test cases
│   │   ├── rubric.md
│   │   └── anti-criteria.md
│   ├── structured_json/
│   │   ├── config.yaml
│   │   ├── inputs/           # 15 test cases
│   │   ├── rubric.md
│   │   └── schema.ts
│   ├── summarization/
│   │   ├── config.yaml
│   │   ├── inputs/           # 10 test cases
│   │   ├── rubric.md
│   │   └── anti-criteria.md
│   ├── multi_step_reasoning/
│   │   ├── config.yaml
│   │   ├── inputs/           # 10 test cases
│   │   └── rubric.md
│   ├── code_generation/
│   │   ├── config.yaml
│   │   ├── inputs/           # 10 test cases
│   │   └── tests/            # Test files per case
│   ├── complex_reasoning/
│   │   ├── config.yaml
│   │   ├── inputs/           # 5 test cases
│   │   └── rubric.md
│   ├── long_doc_analysis/
│   │   ├── config.yaml
│   │   ├── inputs/           # 5 test cases
│   │   └── docs/             # Source documents
│   ├── creative_writing/
│   │   ├── config.yaml
│   │   ├── inputs/           # 5 test cases
│   │   └── rubric.md
│   ├── multimodal/
│   │   ├── config.yaml
│   │   └── inputs/           # 5 test cases + images
│   └── default/
│       ├── config.yaml
│       └── inputs/           # 10 mixed test cases
└── quality-benchmarks.jsonl  # Quality benchmark records (new)
```

### Modified Files

| File | Change |
|------|--------|
| `PAI/Tools/ModelMatrix/index.ts` | Add `quality` field to Benchmark interface, add quality stats |
| `PAI/Tools/inference-matrix.json` | Add quality scores, confidence, and benchmark provenance to each routing entry |
| `PAI/Tools/Inference.ts` | Add `force_model` option to bypass matrix routing during calibration |
| `PAI/Tools/ModelMatrix/README.md` | Document calibration workflow and new commands |
| `~/.claude/PAI/DOCUMENTATION/ARCHITECTURE_SUMMARY.md` | Add Model Benchmark System to architecture summary |

---

## Appendix A: Model Selection Heuristics

When the calibrated matrix routes to a model, these heuristics are applied:

1. **Cost ceiling check** — if best model is `claude/opus` ($15/M input), check if second-best at lower cost is within 5% quality
2. **Latency budget** — for real-time tasks, prefer models with p95 < 500ms even if quality is slightly lower
3. **Token budget awareness** — check current subscription usage via `UsageProbe.ts`; if budget is tight, bias toward free models
4. **Task criticality** — ISC rows marked `critical` always use best-quality model regardless of cost; `nice-to-have` rows always use cheapest adequate model

## Appendix B: Drift Detection Rules

| Condition | Action |
|-----------|--------|
| Quality drops >15% | Flag for manual review, do NOT auto-downgrade routing |
| Quality drops 5-15% | Downgrade routing, send ntfy alert |
| Quality drops <5% | Log only |
| New model >0.9 quality on task | Add as fallback immediately |
| Model >30 days since last benchmark | Flag as stale in dashboard |
| Local model hash changed since last benchmark | Trigger immediate recalibration |

---

*This document serves as both the design specification and the implementation runbook. Update the version and date when significant changes are made. Reference the ModelMatrix README for day-to-day operations.*
