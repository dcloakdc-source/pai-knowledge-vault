---
title: "Consolidate the 4 secret-pattern sources into the shared module"
type: idea
tags: [security, secret-scrubbing, tech-debt, refactor]
created: 2026-05-30
updated: 2026-05-30
quality: 4
source_session: 20260530-105203_scrub-tool-input-secrets
related:
  - slug: harness-spine-rearchitecture
    type: related
---

# Consolidate the 4 secret-pattern sources

## Thesis
There are now **four** secret-pattern definitions, and drift between them is exactly what leaked the xAI key (2026-05-30). They must collapse to one.

## Current state (filed follow-up — advisor 2026-05-30)
- Canonical (new): `hooks/lib/secret-patterns.ts` — `SECRET_PATTERNS` + `redactSecrets`/`hasSecret`/`redactDeep`. Used by the new consumers: `dispatch/PreToolUse.ts` (emit/tracker redaction), `SecretInputScanner.hook.ts`, `redact-log-secrets.ts`.
- Legacy inline (NOT migrated): `hooks/OutputSecretsScanner.hook.ts` (`SECRET_PATTERNS`), `PAI/Tools/SIEM/pai-log-scrubber.ts` (`PATTERNS`), `PAI/Tools/SecretScanQuick.ts` (`PATTERNS`).
- Guard meanwhile: `PAI/Tools/__tests__/secret-pattern-parity.test.ts` asserts every surface (incl. the shared module) covers `REQUIRED_PREFIXES`. This is a band-aid; 4 copies drift worse than 3.

## Implications
- **The migration:** make the 3 legacy files import pattern *sources* from `hooks/lib/secret-patterns.ts` and build their own `RegExp` with their own flags/anchors/replacements (they differ: OutputSecretsScanner uses `g`+marker, pai-log-scrubber uses `g`+`replacement` strings, SecretScanQuick uses `\b`-anchored `.test()` with no `g`). Export pattern *bodies* + names; each file adapts.
- **Risk:** these are enforcement files; preserve each one's exact flag/anchor/replacement semantics or detection changes silently. Do it test-first (the parity test + per-file behavior tests must stay green).
- **Until then:** any new provider prefix MUST be added to all four + the parity `REQUIRED_PREFIXES`.

This is the explicitly-deferred half of [[harness-spine-rearchitecture]]'s "shared-module" follow-up; greenlight separately.
