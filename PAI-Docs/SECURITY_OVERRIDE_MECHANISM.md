# Security Override — Mechanism Reference

> Moved out of force-loaded `CLAUDE.md` (2026-06-11 BPE) — this is *reference* detail needed only when promoting a destructive pattern or debugging the shim layer, not behavior required every turn. The actionable **Security Override Protocol** (variant A/B override steps) stays in `CLAUDE.md`.

## Destructive-op promotion pipeline

Memos describing destructive incidents are auto-extracted by `MemoPromotionExtractor.hook.ts` (PostToolUse), filtered by `PatternValidator.ts` (≥3 literal tokens, ≥2 contextTokens, ≤0.5% corpus hit rate, must match offender), queued at `~/MEMORY/STATE.claude/pending-destructive-patterns.json`, surfaced at SessionStart, and approved via `bun PAI/Tools/MemoryPromoter.ts --apply <slug>` (dual-factor: TOTP + voice-notify confirm code; signed receipts chained in `destructive-promotions.jsonl`). Once in the registry, `DestructiveOpGuard.hook.ts` enforces with override support inside Claude Code. `RegistryAudit.ts` runs lazily on SessionStart (≥24h between runs); patterns with sustained over-rate hit rate auto-demote to `warn_only`.

## Universal coverage layer

Outside Claude Code (terminal, opencode, codex, gemini subprocesses) the same registry is enforced by `pai-guard.ts` PATH shims at `~/.claude/PAI/bin/<tool>`. Each shim is a bash wrapper that execs `bun PAI/Tools/pai-guard.ts --as <tool>` which reads the same `destructive-patterns.json` and `security-overrides.json`. Activate by adding `export PATH="$HOME/.claude/PAI/bin:$PATH"` to `~/.bashrc`. Shims fail-open on registry errors so the user's shell is never broken. Session key for shell-tier overrides is `shell:<ppid>:<tty>` (distinct from Claude Code's session_id).
