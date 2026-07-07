# PAI Functions API — Capabilities

> HTTP gateway on `http://localhost:8890`. No Claude API tokens required.
> Authenticated calls should use a signed PAI identity session.

## For Antigravity
Use `run_in_terminal` with the commands below. The service runs 24/7 as a systemd unit.

## For Ollama / Scripts
Use Python urllib or curl. Same endpoints, same JSON.

---

## Authorization model

| Endpoint class | Requirement |
| --- | --- |
| `OPTIONS`, `/health`, `GET /v1/models` | Public |
| `/ws` | Valid PAI session id or JWT with `functions:read` |
| `/v1/cred*` | Identity session or OIDC proof satisfying `functions:cred`; currently requires `oidc_fresh` |
| Memory and DB write/delete routes | Identity session or OIDC/DID proof satisfying `functions:write` |
| `POST /v1/ingest/social` | Identity session or OIDC/DID proof satisfying `functions:write` |
| Static mutation/dispatch routes listed below | Identity session or OIDC/DID proof satisfying `functions:write` |
| Most other `/v1/*`, `/api/*`, `/ui/*` | Identity session or OIDC/DID proof satisfying `functions:read` |

Use `FunctionsAPIClient.ts` for protected examples. Raw curl examples below are illustrative payloads only unless they include a `PAI-Session` or `X-PAI-Identity-Session` header.

Legacy `PAI_API_KEY` bearer or `X-PAI-Key` authentication is read-only compatibility. It can satisfy `functions:read`, but it cannot satisfy `functions:write`, `functions:cred`, or fresh/MFA proof requirements.

Static `functions:write` routes:

- `POST /v1/chat/completions`
- `POST /v1/db/memory`
- `POST /v1/db/state`
- `POST /v1/debate`
- `POST /v1/diagrams/excalidraw`
- `POST /v1/distill`
- `POST /v1/gemini`
- `POST /v1/harvest`
- `POST /v1/icm/store`
- `POST /v1/memory/write`
- `POST /v1/mercury-chat`
- `POST /v1/mercury-fim`
- `POST /v1/metrics/push`
- `POST /v1/opai/complete`
- `POST /v1/opai/dequeue`
- `POST /v1/opai/enqueue`
- `POST /v1/owui/push`
- `POST /v1/pai-pi`
- `POST /v1/schedule`
- `POST /v1/voice`
- `DELETE /v1/db/memory/:id`
- `DELETE /v1/schedule/:id`

Body-dependent `functions:write` route:

- `POST /v1/research` only when `save: true`

Dynamic route IDs are constrained before filesystem or subprocess use:

- `GET /v1/debate/:id` accepts generated debate IDs shaped like `debate-YYYYMMDD-abcdef`.
- `DELETE /v1/schedule/:id` accepts one safe token of letters, numbers, `.`, `_`, `:`, or `-`; traversal and slashes are rejected.

OPAI/PNK (PAI OpenCode) queue inputs are constrained before `OpaiQueue.ts` subprocess calls:

- `POST /v1/opai/enqueue` accepts `priority` values `low`, `normal`, or `high`; `source` must be a short safe token; `task` and `context` reject control characters and oversized payloads.
- `POST /v1/opai/complete` accepts `outcome` values `success`, `failed`, or `abandoned`.

Schedule creation inputs are constrained before `Scheduler.ts` subprocess calls:

- `POST /v1/schedule` accepts five-field cron expressions only. `command` and `label` reject control characters and oversized payloads; `label` is restricted to safe display-token characters.

Debate creation inputs are constrained before shared-inbox file writes:

- `POST /v1/debate` requires a bounded `question`; optional `context` and `claude_position` reject control characters and oversized payloads. `positions_timeout_ms` is clamped to a bounded polling range.

ICM store inputs are constrained before `icm store` subprocess calls:

- `POST /v1/icm/store` accepts short safe-token topics only; `importance` is restricted to `low`, `medium`, `high`, or `critical`; `content` must be non-empty, bounded, and free of non-whitespace control characters.

---

## Identity session authentication

Preferred auth header:

```bash
Authorization: PAI-Session <token>
```

Browser or proxy clients may use:

```bash
X-PAI-Identity-Session: <token>
```

Issue and verify a short-lived session with the CLI helper:

```bash
bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts whoami \
  --uid USR-001 \
  --proof-level registry \
  --ttl-sec 300 \
  --revoke-after
```

Use an existing session token without printing it:

```bash
PAI_IDENTITY_SESSION_TOKEN="<token>" \
  bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts whoami
```

Call another endpoint with the same session auth:

```bash
PAI_IDENTITY_SESSION_TOKEN="<oidc-fresh-or-better-session>" \
  bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --path /v1/cred \
  --body '{"service":"PAI_API_KEY","ttl":300}'
```

The helper never prints the raw session token. It reports only the session id for audit lookup.
Local `--uid` issuance is registry proof only. Fresh/MFA sessions must be minted from verified OIDC/MAS proof.

MAS fresh-session flow:

```bash
bun ~/.claude/PAI/Tools/Identity/MASClient.ts fresh-session-device \
  --client-id 01KV4WY6EWQ2XANBPNJJGMGNDE \
  --token-out /tmp/pai-mas-session.json \
  --policy fresh \
  --ttl-sec 300
```

Open the returned MAS device URL, enter the code, and complete the MAS login/continue page. After the command succeeds, use the session file:

```bash
bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --session-file /tmp/pai-mas-session.json \
  --path /v1/cred/vault/status
```

Session lifecycle controls:

```bash
# List active, unexpired sessions
bun ~/.claude/PAI/Tools/Identity/SessionIdentity.ts list

# Include revoked and expired metadata
bun ~/.claude/PAI/Tools/Identity/SessionIdentity.ts list --all

# Revoke one session
bun ~/.claude/PAI/Tools/Identity/SessionIdentity.ts revoke --session-id ids-...

# Revoke all active sessions for an identity
bun ~/.claude/PAI/Tools/Identity/SessionIdentity.ts revoke --uid USR-001

# Remove expired session metadata after audit logging
bun ~/.claude/PAI/Tools/Identity/SessionIdentity.ts purge-expired
```

Session classes:

- Human sessions are minted from verified OIDC/MAS proof or local registry proof for diagnostics.
- Agent/service sessions are local registry proof only unless a future delegation credential explicitly upgrades them.
- `whoami` reports `subject_class` for identity-session callers and `legacy_read_only` for API-key compatibility callers.

Identity security report:

```bash
bun ~/.claude/PAI/Tools/Identity/IdentitySecurityReport.ts \
  --since-hours 24 \
  --samples 5
```

The report summarizes sessions, proof-level usage, identity proof decisions, authorization decisions, and credential-access audit results. It does not print raw session tokens, token hashes, or credential values.

---

## research
**POST /v1/research** — YouTube search + Ollama AI summarization.

**Auth:** `functions:read`; `save: true` requires `functions:write`.

**Parameters:**
- `topic`: string — what to research
- `depth`: "quick" | "standard" (default: standard)
- `save`: boolean — save results to MEMORY/Research/ (default: false)

**Example:**
```bash
bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --uid USR-001 \
  --path /v1/research \
  --body '{"topic":"whisper speech recognition","depth":"quick"}' \
  --revoke-after
```

## voice
**POST /v1/voice** — Speak a message through the PAI voice compatibility path. This route currently forwards through Pulse at `http://localhost:31337/notify`; Pulse then forwards to the configured voice backend. Voice runtime is migrating to the Legion server, so callers should use this route or Pulse rather than binding directly to a TTS host. Optionally cast to Office display.

**Auth:** `functions:write`

**Parameters:**
- `message`: string — text to speak
- `cast`: boolean — cast to Office display speaker (default: false)
- `voice_id`: string — configured PAI voice id/name (default resolved by Pulse/VoiceServer)
- `voice_enabled`: boolean — set `false` for routing tests without speech (default: true)

**Example:**
```bash
bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --uid USR-001 \
  --path /v1/voice \
  --body '{"message":"Research complete","cast":true}' \
  --revoke-after
```

## diagrams_excalidraw
**POST /v1/diagrams/excalidraw** — Generate an Excalidraw diagram artifact.

**Auth:** `functions:write`

**Parameters:**
- `title`: string — diagram title
- `type`: string — diagram type, default `flowchart`
- `open`: boolean — open generated artifact, default true
- `mermaid`: string — optional Mermaid source

**Example:**
```bash
bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --uid USR-001 \
  --path /v1/diagrams/excalidraw \
  --body '{"title":"System Flow","type":"flowchart","open":false}' \
  --revoke-after
```

## owui_push
**POST /v1/owui/push** — Create an Open WebUI chat notification using the configured OWUI admin token.

**Auth:** `functions:write`

**Parameters:**
- `message`: string — message to place in the new chat
- `title`: string — chat title, default `PAI Notification`

**Example:**
```bash
bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --uid USR-001 \
  --path /v1/owui/push \
  --body '{"title":"PAI Notice","message":"Research complete"}' \
  --revoke-after
```

## mercury
**POST /v1/mercury-fim** and **POST /v1/mercury-chat** — Call Inception Mercury FIM/chat models.

**Auth:** `functions:write`

These routes use `MERCURY_API_KEY` or `INCEPTION_API_KEY` and consume external model quota.

## chat_completions
**POST /v1/chat/completions** — OpenAI-compatible chat completion endpoint for PAI models.

**Auth:** `functions:write`

This route may run local Ollama inference or cloud-backed PAI inference depending on `model`.

## memory_list
**GET /v1/memory** — List files under a PAI memory path.

**Auth:** `functions:read`

Search snippets use the same memory path restrictions as direct reads. Indexed files with parent-directory escapes, symlink escapes, or secret-bearing paths are skipped instead of previewed.

**Parameters:**
- `path`: string — relative path under ~/.claude/ (default: MEMORY). Parent-directory escapes and secret-bearing paths are blocked.

**Example:**
```bash
bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --uid USR-001 \
  --path '/v1/memory?path=MEMORY/Research' \
  --revoke-after
```

## memory_read
**GET /v1/memory/read** — Read a PAI memory file as text.

**Auth:** `functions:read`

**Parameters:**
- `file`: string — relative path under ~/.claude/. Parent-directory escapes, symlink escapes, and secret-bearing paths are blocked.

**Example:**
```bash
bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --uid USR-001 \
  --path '/v1/memory/read?file=MEMORY/LEARNING/REFLECTIONS/algorithm-reflections.jsonl' \
  --revoke-after
```

## ui_file
**GET /ui/file** — Browser-friendly read-only file viewer.

**Auth:** `functions:read`

Uses the same path restrictions as `memory_read`: parent-directory escapes, symlink escapes, and secret-bearing paths are blocked. Rendered file names, paths, errors, and content are HTML-escaped.

**Example:**
```bash
bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --uid USR-001 \
  --path '/ui/file?path=PAI/FUNCTIONS_API.md' \
  --revoke-after
```

## memory_write
**POST /v1/memory/write** — Write content to a PAI memory file.

**Auth:** `functions:write`.

**Parameters:**
- `file`: string — relative path under ~/.claude/. Parent-directory escapes, symlink escapes, directories, and secret-bearing paths are blocked.
- `content`: string — file contents
- `append`: boolean — append instead of overwrite (default: false)

**Example:**
```bash
bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --uid USR-001 \
  --path /v1/memory/write \
  --body '{"file":"MEMORY/Research/notes.md","content":"# My Notes\n..."}' \
  --revoke-after
```

## memory_search
**POST /v1/memory/search** — Semantic search across PAI memory using vector embeddings (Ollama mxbai-embed-large).

**Auth:** `functions:read`

**Parameters:**
- `query`: string — natural language search query
- `limit`: number — max results to return (default: 10)

**Example:**
```bash
bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --uid USR-001 \
  --path /v1/memory/search \
  --body '{"query":"voice server audio","limit":5}' \
  --revoke-after
```

## cred — Just-In-Time credential access

**POST /v1/cred** — Issue a time-limited credential lease. Fetches from `~/.config/PAI/secrets.env` or env vars. Every access logged to `MEMORY/SECURITY/cred-access.jsonl`. Credential values are never logged.

**Auth:** `functions:cred`; requires fresh OIDC/MAS proof (`oidc_fresh`) and route-level agent authorization where applicable.

For agent identity sessions, `X-Agent-Id` must match the authenticated agent UID. Human/operator sessions may provide an explicit `X-Agent-Id` for the agent context they are administering.

**Parameters:**
- `service`: string — credential name (e.g. `ANTHROPIC_API_KEY`, `github/token`). Allowed characters: letters, numbers, `_`, `.`, `/`, `:`, `@`, `-`; traversal, control characters, whitespace, leading `-`, backslashes, and names over 128 characters are rejected before backend lookup.
- `ttl`: number — lease duration in seconds (default: 900, min: 60, max: 86400)

**Response:** `{ lease_id, service, backend, credential, issued_at, expires_at, ttl_seconds }`

**Example:**
```bash
PAI_IDENTITY_SESSION_TOKEN="<oidc-fresh-or-better-session>" \
  bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --path /v1/cred \
  --agent-id AGT-001 \
  --body '{"service":"ANTHROPIC_API_KEY","ttl":300}'
```

**GET /v1/cred/leases** — List all active (non-expired) leases. Credential values are omitted. Requires `X-Agent-Id` for a registered `pai-admin` agent.

```bash
PAI_IDENTITY_SESSION_TOKEN="<oidc-fresh-or-better-session>" \
  bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --path /v1/cred/leases \
  --agent-id AGT-001
```

**DELETE /v1/cred/lease/{id}** — Revoke an active lease. Requires `X-Agent-Id` for a registered `pai-admin` agent.

```bash
PAI_IDENTITY_SESSION_TOKEN="<oidc-fresh-or-better-session>" \
  bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --method DELETE \
  --path /v1/cred/lease/686cbb324e564a88a1fd55074de0663b \
  --agent-id AGT-001
```

**Setup — add secrets to `~/.config/PAI/secrets.env` (chmod 600):**
```bash
echo 'ANTHROPIC_API_KEY=sk-ant-...' >> ~/.config/PAI/secrets.env
echo 'GITHUB_TOKEN=ghp_...' >> ~/.config/PAI/secrets.env
```

## cred/vault — Vaultwarden integration

**GET /v1/cred/vault/status** — Check Vaultwarden reachability and session state.

**Auth:** `functions:cred`; requires fresh OIDC/MAS proof.

```bash
PAI_IDENTITY_SESSION_TOKEN="<oidc-fresh-or-better-session>" \
  bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --path /v1/cred/vault/status
# → { reachable, status, email, session_active, session_expires_at }
```

**POST /v1/cred/unlock** — Unlock vault and cache session (14-min TTL).

**Auth:** `functions:cred`; requires fresh OIDC/MAS proof and `X-Agent-Id` for a registered `pai-admin` agent.

Three modes:
- `{session}` — inject existing BW_SESSION token directly (no bw unlock call)
- `{password}` — unlock with explicit password
- `{}` — auto-read `BW_MASTER_PASSWORD` from `~/.config/PAI/secrets.env`

```bash
# Auto-unlock (BW_MASTER_PASSWORD must be in secrets.env)
PAI_IDENTITY_SESSION_TOKEN="<oidc-fresh-or-better-session>" \
  bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --path /v1/cred/unlock \
  --agent-id AGT-001 \
  --body '{}'

# Manual session injection
PAI_IDENTITY_SESSION_TOKEN="<oidc-fresh-or-better-session>" \
  bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --path /v1/cred/unlock \
  --agent-id AGT-001 \
  --body '{"session":"<bw session token>"}'
```

Response: `{ ok, session_expires_at }` — session token never returned.

**POST /v1/cred/rotate** — Rotate a credential.

**Parameters:**
- `service`: string — credential name (same as `/v1/cred`)
- `strategy`: `"vault"` | `"source"` | `"both"`
- `ttl`: number — TTL for the new lease issued after vault rotation (default: 900)

Strategies:
- **vault**: generates new 32-char password via `bw generate`, updates vault item, revokes all active leases, returns fresh lease with new credential
- **source**: POSTs `{service, rotated_at}` to `{SERVICE}_ROTATE_URL` from secrets.env
- **both**: vault rotation then source notification

```bash
# Rotate in vault only (requires unlocked session)
PAI_IDENTITY_SESSION_TOKEN="<oidc-fresh-or-better-session>" \
  bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --path /v1/cred/rotate \
  --body '{"service":"GITHUB_TOKEN","strategy":"vault"}'

# Notify source service only
PAI_IDENTITY_SESSION_TOKEN="<oidc-fresh-or-better-session>" \
  bun ~/.claude/PAI/Tools/FunctionsAPIClient.ts request \
  --path /v1/cred/rotate \
  --body '{"service":"GITHUB_TOKEN","strategy":"source"}'
```

**Setup for rotation webhook:**
```bash
echo 'GITHUB_TOKEN_ROTATE_URL=https://api.github.com/...' >> ~/.config/PAI/secrets.env
```

**Setup for auto-unlock:**
```bash
echo 'BW_MASTER_PASSWORD=your-vault-password' >> ~/.config/PAI/secrets.env
```

Credential lookup priority: `Vaultwarden → secrets.env → env-var`. Vault is tried silently — if unreachable or locked, falls back automatically.

---
*Updated 2026-06-16 — documented identity-session auth requirements and credential fresh-proof examples*
