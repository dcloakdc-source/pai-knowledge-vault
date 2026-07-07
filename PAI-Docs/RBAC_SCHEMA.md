# PAI RBAC Schema — SINA Initiative
*Source of truth for agent identity tiers, scopes, and credential governance.*
*Last updated: 2026-04-14 (SINA ISC-1 through ISC-4)*

---

## Tier Model

| Tier | Name | Who | Scope |
|------|------|-----|-------|
| **Tier 0** | Sovereign | USR-001 (Duane), AGT-001 (PNC) | Full system control — all secrets, all writes, all agent tasking |
| **Tier 1** | Privileged | AGT-002 (PNG), AGT-003 (PNK / opai), AGT-005 (Antigravity), AGT-008 (PNX / Codex) | Production secrets in allow-list, memory write, cross-agent tasking |
| **Tier 2** | Standard | AGT-004 (MCP Server), AGT-007 (PNO / Ollama), ephemeral agents | Memory read-only, no secret access, no cross-agent write |

---

## Tier 0 — Sovereign Scope

**Agents:** `USR-001`, `AGT-001`

| Permission | Details |
|-----------|---------|
| `secret:*` | Full Vaultwarden access via `fetch-secret.sh` |
| `memory:write` | Write to any MEMORY/ path |
| `memory:read` | Read any MEMORY/ path |
| `agent:task` | Task any agent via inbox |
| `agent:revoke` | Revoke any agent's access |
| `jit:issue` | Issue JIT credentials to Tier 1/2 agents |
| `jit:revoke` | Revoke any active JIT token |
| `rbac:admin` | Modify `agents.yml` and `RBAC_SCHEMA.md` |
| `system:restart` | Restart PAI services |

---

## Tier 1 — Privileged Scope

**Agents:** `AGT-002` (PNG), `AGT-003` (PNK / opai), `AGT-005` (Antigravity), `AGT-008` (PNX / Codex)

| Permission | Details |
|-----------|---------|
| `secret:listed` | Only secrets in `agents.yml → allowed_secrets` |
| `memory:write` | Write to own WORK/, LEARNING/, STATE/ paths |
| `memory:read` | Read any non-sensitive MEMORY/ path |
| `agent:task` | Send PoW-signed tasks to other Tier 1/2 agents (not Tier 0 tasking) |
| `jit:request` | Request JIT credentials from Tier 0 for specific ops |
| `jit:use` | Use active JIT token (time-limited, single-use) |
| `exec:bash` | Execute bash commands within project boundaries (opai specific) |

---

## Tier 2 — Standard Scope

**Agents:** `AGT-004` (MCP Server), `AGT-007` (PNO / Ollama), `AGT-EXT-*` (external ephemeral agents)

| Permission | Details |
|-----------|---------|
| `memory:read` | Read from public MEMORY/ paths only (no SECURITY/, no STATE/pki/) |
| `agent:observe` | Read-only inbox subscription — no write access |

---

## JIT Credential Issuance Process (sec:jit) — ISC-2

JIT credentials allow Tier 1 agents to temporarily access secrets outside their normal allow-list.

### Flow

```
Agent (Tier 1)                    Tier 0 (PNC / USR-001)
     │                                     │
     │── jit-request.json ──→ claude/ ─────│  (PoW-signed, via inbox)
     │   { secret, reason, ttl_minutes }   │
     │                                     │── validates: agent tier, reason, ttl ≤ 60min
     │                                     │── calls: bun PAI/Tools/jit-cred.ts --issue
     │                                     │── writes: MEMORY/STATE/jit-tokens/{id}.json
     │←── jit-grant.json ──── gemini/ ─────│  (includes token_id, expires_at)
     │                                     │
     │── fetch-secret.sh <secret> --jit {token_id}
     │   (fetch-secret.sh validates token, returns secret, marks token USED)
```

### JIT Token Schema (`MEMORY/STATE/jit-tokens/{id}.json`)

```json
{
  "token_id": "JIT-{timestamp}-{random}",
  "agent_id": "AGT-002",
  "secret_name": "SOME_SECRET",
  "reason": "One-line justification",
  "issued_by": "USR-001",
  "issued_at": "ISO timestamp",
  "expires_at": "ISO timestamp (max +60 min)",
  "used": false,
  "used_at": null
}
```

### Constraints
- Max TTL: 60 minutes
- Single-use: `fetch-secret.sh` marks `used: true` on first retrieval
- Tier 0 approval required: Duane or PNC must explicitly write the grant file
- Audit: All JIT issuances logged to `MEMORY/SECURITY/jit-audit.jsonl`
- Denial: Requests for secrets marked `tier0-only` are auto-denied

---

## Secret Storage Strategy (agent-to-agent auth) — ISC-3

| Secret Type | Storage | Access Pattern |
|-------------|---------|----------------|
| Production secrets (API keys, DB passwords) | Vaultwarden (encrypted vault) | `fetch-secret.sh` RBAC proxy only |
| Runtime session vars (PAI_API_KEY, OLLAMA_URL) | `~/.config/PAI/secrets.env` | Loaded at service start; never in inbox |
| Agent-to-agent auth | PoW challenge-response + JWT (PKI) | No shared secrets — cryptographic only |
| JIT tokens | `MEMORY/STATE/jit-tokens/` (gitignored) | Single-use, time-limited, in-memory |
| Short-lived session creds | `MEMORY/STATE/session-creds/` (gitignored, 600) | Written by jit-cred.ts, auto-deleted on expiry |

### Agent-to-Agent Auth Rule
**Agents NEVER pass secrets in inbox messages.** Authentication between agents uses:
1. **PoW** (`AgentPoW`) — proves sender identity, prevents replay
2. **JWT** — for Command Center API calls (PKI-issued, EdDSA-signed)
3. **JIT tokens** — for escalated one-time secret access

---

## Identity Registry — Participating SINA Agents (ISC-4)

| UID | Name | Role | Tier | Inbox Path |
|-----|------|------|------|------------|
| `USR-001` | Duane | Human principal | Tier 0 | (direct) |
| `AGT-001` | PAI Nova Claude (PNC) | Primary AI, architect | Tier 0 | `agent-inbox/claude/` |
| `AGT-002` | PAI Nova Gemini (PNG) | Research, network ops | Tier 1 | `agent-inbox/gemini/` |
| `AGT-003` | PAI Nova OpenCode (PNK / opai) | Autonomous coding, automation | Tier 1 | `agent-inbox/opencode/` |
| `AGT-004` | PAI MCP Server | Tool exposure | Tier 2 | (no inbox — stdio only) |
| `AGT-005` | PAI Antigravity (Gemini CLI) | Alternative PNG interface | Tier 1 | `agent-inbox/antigravity/` |
| `AGT-007` | PAI Nova Ollama (PNO) | Local classification, JSON, summarization | Tier 2 | (local inference service) |
| `AGT-008` | PAI Nova Codex (PNX) | Cato adversary, Forge worker, autonomous code execution | Tier 1 | `agent-inbox/codex/` |

### SINA Coordination Roles
- **Orchestrator/Architect:** `AGT-001` (PNC) — RBAC decisions, Tier 0-1 governance
- **Network Specialist:** `AGT-002` / `AGT-005` (PNG) — DNS, SSH, asset discovery
- **Autonomous Developer:** `AGT-003` (PNK / opai) — Legacy cleanup, refactoring, batch tasks
- **Cross-vendor Auditor / Forge Worker:** `AGT-008` (PNX / Codex) — Cato adversarial review and high-reasoning code generation
- **Auditor (optional):** `AGT-004` (MCP) — compliance cross-reference (read-only)
- **Local Triage:** `AGT-007` (PNO / Ollama) — Local classification, extraction, summarization

---

## Enforcement Points

| Location | What It Enforces |
|----------|-----------------|
| `PAI/Tools/fetch-secret.sh` | RBAC allow-list before Vaultwarden proxy |
| `PAI/Tools/jit-cred.ts` | JIT token issuance + expiry + single-use |
| `PAI/Tools/AgentPoW.ts` | PoW signing + verification for all inbox messages |
| `PAI/agents.yml` | Canonical agent registry (role + allowed_secrets) |
| `hooks/SecurityValidator.hook.ts` | Runtime enforcement at tool call boundary |

---

*Run `bun PAI/Tools/RBACSecurityAudit.ts` to validate all agents against this schema.*
