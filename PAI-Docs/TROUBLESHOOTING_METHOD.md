# PAI Troubleshooting Method

> Cross-engine protocol for visible, iterative troubleshooting. All PAI engines (PNC, PNX, PNK) load this via their AGENTS.md / CLAUDE.md Identity Anchors section. PNG (Antigravity) and PNO (Ollama) are research/inference-only and skip this protocol.

## Core Principle

Troubleshooting is an iterative loop, not a linear path. The value is in seeing the iterations — the failed attempts, the hypotheses, the course corrections. A session that shows only the final success is hiding the method.

## The Loop

```
1. SHOW FAILING STATE → 2. STATE HYPOTHESIS → 3. TRY ONE FIX → 4. RE-PROBE → 5. ITERATE OR ESCALATE
```

### Step 1 — Show the Failing State

Run the minimum probe that reveals the problem. Show the full output.

```
🔍 PROBE: <the exact command or tool call>
📄 OUTPUT: <full stdout/stderr — do not truncate the error>
💥 EXPECTED vs ACTUAL: <one-sentence delta>
```

**Rule:** A troubleshooting session that begins without a visible failing state is guessing.

### Step 2 — State the Hypothesis

One sentence. What you think is wrong and why.

```
💡 HYPOTHESIS: <because X, the problem is Y>
```

**Rule:** If you cannot state the hypothesis in one sentence, you haven't narrowed enough. Re-probe to gather more signal.

### Step 3 — Try One Fix

The smallest possible change. Show the exact edit or command.

```
🔧 FIX: <the exact change — one line, one config knob, one file>
```

**Rule:** One change per iteration. Multiple changes = you won't know which one fixed it.

### Step 4 — Re-Probe

Run the EXACT SAME probe from Step 1. Show the result.

```
🔍 RE-PROBE: <same command>
📄 OUTPUT: <new result>
✅ STATUS: [FIXED | SAME | WORSE | DIFFERENT ERROR]
```

**Rule:** A different probe invalidates the comparison. Always re-run the same command.

### Step 5 — Iterate or Escalate

- **FIXED:** Document what was learned.
- **SAME or WORSE:** The new state is Step 1 for the next loop. Do NOT re-state the original failing state — state the *current* failing state.
- **DIFFERENT ERROR:** The fix broke something else. Revert it, or state the new error as Step 1.

## Integration with the Algorithm

The Troubleshooting Method lives inside the Algorithm phases, not beside them:

| Algorithm Phase | Troubleshooting applies when... |
|---|---|
| **OBSERVE** | Gate A (Diagnostic) fires. Reproduce-First captures the failing state. |
| **THINK** | Hypothesis is formed from the reproduction evidence. Premortem accounts for the fix not working. |
| **PLAN** | Each iteration is one PLan item. One fix per cycle. |
| **BUILD/EXECUTE** | The Fix is applied. Re-probe runs immediately after. |
| **VERIFY** | The final re-probe IS the verification. Inline verification captures the iteration chain. |
| **LEARN** | The learning from the troubleshooting loop is routed via the Learning Router. |

## ISA Troubleshooting Log

For E2+ troubleshooting sessions (Extended effort or higher), capture the iteration chain in the ISA:

```
## Troubleshooting Log

### T1: <what was tried>
- **Probe:** <command>
- **Output:** <error>
- **Hypothesis:** <one sentence>

### T2: <one change>
- **Probe:** <same command>
- **Output:** <new result>
- **Status:** FIXED | SAME | WORSE | DIFFERENT ERROR
- **Hypothesis:** <updated or confirmed>

### Resolution
- **Fix:** <what finally worked>
- **Root cause:** <why the fix worked>
- **Learning:** <for Knowledge or feedback memory>
```

**Skip conditions:** E1 Standard tasks skip the log — the overhead doesn't justify the value. E2+ debugging sessions MUST include it. Pure-additive feature work with no troubleshooting skips it.

## Engine-Specific Notes

| Engine | Verbosity | Notes |
|---|---|---|
| **PNC (Claude Code)** | `verbose: true` in settings.json | Tool calls shown with full parameters. Post-compaction preserves phase output. |
| **PNX (Codex)** | `--verbose` flag for `codex exec` | Sandbox mode suppresses intermediate output — use `--verbose` to expose it. |
| **PNK (OpenCode)** | Inherent — TUI streams tool calls | Platform-native visibility. Use the ISA Troubleshooting Log for persistence. |

## Why This Works

The loop is resistant to the two failure modes that silence troubleshooting:

1. **Premature conclusion:** Stating the hypothesis in one sentence forces narrowing before fixing. If you can't state it, you gather more signal.
2. **Silent retry:** Showing each re-probe output prevents "I fixed it" without evidence. The failed attempts are preserved in the log — they are the method, not the problem.
