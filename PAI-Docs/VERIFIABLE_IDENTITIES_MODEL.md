# PAI Verifiable Identities Model

Source artifacts:
- `/home/duane/.claude/PAI/IAM_ARCHITECTURE.md`
- `/home/duane/.claude/PAI/RBAC_SCHEMA.md`
- `/home/duane/.claude/PAI/RBAC_ORG_STRUCTURES_MODEL.md`
- `/home/duane/.claude/PAI/config/agents.yml`
- `/home/duane/.claude/PAI/config/inventory.yml`
- `/home/duane/.claude/PAI/Tools/PKI/`
- `/home/duane/.claude/PAI/Tools/AgentPoW.ts`
- `/home/duane/.claude/PAI/Tools/InboxWatchdog.ts`

## Purpose

PAI already has persistent UIDs, IAM tiers, Authentik groups, Vaultwarden/JIT secrets, Ed25519 JWTs, SSH/TLS certificate plans, mailbox HMAC signatures, and PoW-gated inter-agent messages.

The gap: not every proof is bound to the same canonical identity. Some JWT subjects are loose strings such as `claude-agent`; inbox messages may claim `from: PNX`; HMAC proves possession of a shared secret, not which UID signed; PoW throttles loops but does not prove authorization.

This model makes every actor identity verifiable by binding:

```text
UID -> DID URI -> key material -> credential -> proof -> RBAC subject -> audit event
```

## Identity Classes

| Class | UID Prefix | DID Form | Registry |
|---|---|---|---|
| Human principal | `USR-###` | `did:pai:usr:001` | Principal registry / Authentik |
| Agent or engine | `AGT-###` | `did:pai:agt:001` | `config/agents.yml` |
| System/node | `SYS-###` | `did:pai:sys:001` | `config/inventory.yml` |
| Asset/service/device | `AST-###` | `did:pai:ast:001` | `config/inventory.yml` / service registry |
| Account | `ACC-###` | `did:pai:acc:001` | `config/account-structure.yml` |
| Org session | `ORG-...` | `did:pai:org:{slug}` | `MEMORY/STATE/org-sessions/` |
| Org role assignment | `ORA-...` | `did:pai:ora:{id}` | `MEMORY/STATE/org-sessions/` |

Rules:

- A DID never replaces the UID; it is the verifiable identity URI for that UID.
- Every DID resolves locally to a DID document.
- Every DID document must include the UID, current lifecycle state, public keys, issuer, registry source, and allowed credential types.
- Aliases like `PNX`, `Codex`, `Forge`, or `claude-agent` are display aliases only. Authorization keys on UID/DID.

## DID Document Shape

Canonical storage:

- Registry: `~/.claude/PAI/config/verifiable-identities.yml`
- Runtime DID documents: `MEMORY/STATE/identities/{uid}.did.json`
- Revocation list: `MEMORY/SECURITY/identity-revocations.jsonl`
- Audit log: `MEMORY/SECURITY/identity-proof-audit.jsonl`

Example:

```json
{
  "id": "did:pai:agt:003",
  "uid": "AGT-003",
  "aliases": ["PNK", "opai", "opencode"],
  "class": "agent",
  "registry_source": "/home/duane/.claude/PAI/config/agents.yml",
  "host_uid": "SYS-001",
  "rbac_tier": "T1",
  "status": "active",
  "issuer": "did:pai:svc:pai-pki",
  "created_at": "2026-06-14T00:00:00Z",
  "updated_at": "2026-06-14T00:00:00Z",
  "verification_methods": [
    {
      "id": "did:pai:agt:003#jwt-ed25519-2026q2",
      "type": "JsonWebKey2020",
      "controller": "did:pai:agt:003",
      "publicKeyJwkRef": "~/.config/PAI/pki/public.jwk.json",
      "purpose": ["authentication", "assertionMethod"]
    },
    {
      "id": "did:pai:agt:003#mailbox-hmac-2026q2",
      "type": "PAI-HMAC-SHA256",
      "controller": "did:pai:agt:003",
      "secretRef": "Vaultwarden:mailbox/AGT-003",
      "purpose": ["messageAuthentication"]
    }
  ],
  "credentials": [
    {
      "type": "PAIAgentIdentityCredential",
      "credential_id": "cred:pai:agt:003:2026q2",
      "status": "active",
      "expires_at": "2026-09-14T00:00:00Z"
    }
  ]
}
```

## Credential Types

### 1. PAI Identity Credential

Longer-lived credential proving a UID exists and is controlled by a current key.

JWT claims:

```json
{
  "iss": "did:pai:svc:pai-pki",
  "sub": "did:pai:agt:003",
  "jti": "cred:pai:agt:003:2026q2",
  "uid": "AGT-003",
  "class": "agent",
  "aliases": ["PNK", "opai"],
  "host_uid": "SYS-001",
  "rbac_tier": "T1",
  "credential_type": "PAIAgentIdentityCredential",
  "scope": "identity:prove agent:message memory:write",
  "iat": 1781455200,
  "exp": 1789231200
}
```

Use:

- Login/session bootstrap.
- Agent/service registration.
- Mapping human-readable aliases to canonical UID/DID.
- Establishing the identity ceiling for RBAC.

### 2. PAI Session Credential

Short-lived credential proving an actor is active in this session.

JWT claims:

```json
{
  "iss": "did:pai:svc:pai-pki",
  "sub": "did:pai:agt:003",
  "uid": "AGT-003",
  "session_id": "SES-20260614-164114",
  "credential_type": "PAISessionCredential",
  "scope": "agent:message task:update memory:write",
  "iat": 1781455274,
  "exp": 1781458874
}
```

Use:

- API calls.
- Team/session activity.
- Runtime audit events.

TTL target: 1 hour for agents, shorter for ephemeral external agents.

### 3. PAI Org Role Credential

Short-lived credential proving a UID holds one org role in one org session.

JWT claims:

```json
{
  "iss": "did:pai:svc:pai-pki",
  "sub": "did:pai:ora:001",
  "holder": "did:pai:agt:003",
  "uid": "AGT-003",
  "org_session": "ORG-20260614-rbac-example",
  "structure": "business",
  "role": "Engineer",
  "credential_type": "PAIOrgRoleCredential",
  "grants": ["task:update:owned", "memory:write:work"],
  "resource_scope": ["TASK-001"],
  "iat": 1781455274,
  "exp": 1781458874
}
```

Use:

- Org-structure RBAC.
- Separation-of-duty checks.
- Proving an action was done as a scoped role, not global identity.

### 4. PAI Message Proof

Envelope around agent messages:

```json
{
  "id": "MSG-...",
  "type": "MSG",
  "from_uid": "AGT-003",
  "from_did": "did:pai:agt:003",
  "to_uid": "AGT-001",
  "to_did": "did:pai:agt:001",
  "session_id": "SES-20260614-164114",
  "body_sha256": "...",
  "created": "2026-06-14T16:41:14Z",
  "pow": {
    "challenge": "...",
    "sender_id": "AGT-003",
    "nonce": 12345
  },
  "proof": {
    "type": "Ed25519Signature2020",
    "verification_method": "did:pai:agt:003#jwt-ed25519-2026q2",
    "signature": "base64url..."
  }
}
```

Use:

- Inter-agent inbox and Switchboard messages.
- Replacement path for shared HMAC mailbox signatures.

PoW remains rate-limit/economic-stake proof. It is not identity proof.

## Proof Families

| Proof | Existing Artifact | Identity Meaning | Required Upgrade |
|---|---|---|---|
| Ed25519 JWT | `Tools/PKI/issue.ts`, `verify.ts` | Issuer signed claims | Make `sub` a DID and add `uid`, `credential_type`, `rbac_tier`, `kid`. |
| SSH certificate | `Tools/PKI/issue-ssh-cert.ts`, `ssh-ca.pub` | Host/admin access | Principal must include UID/DID; cert extensions include purpose and TTL. |
| TLS certificate | step-ca / cert manager strategy | Service endpoint identity | SAN includes service DID or UID DNS label where feasible. |
| Mailbox signature | `MailboxSig` via `InboxWatchdog.ts` | HMAC possession | Migrate to per-UID key or Ed25519 message proof. |
| PoW | `AgentPoW.ts` | Anti-spam / anti-loop | Keep as required companion proof, not auth proof. |
| Authentik group | IAM docs | Human/service group membership | Treat as attribute source, not cryptographic proof alone. |

## Trust Anchors

| Anchor | Location | Purpose |
|---|---|---|
| PAI identity issuer | `~/.config/PAI/pki/signing.jwk.json` | Issues Ed25519 JWT credentials. |
| PAI public verifier | `~/.config/PAI/pki/public.jwk.json` | Verifies Ed25519 JWT credentials. |
| SSH CA | `~/.config/PAI/pki/ssh-ca` and `ssh-ca.pub` | Issues SSH certs. |
| TLS CA | step-ca on PAI infra | Issues service transport certs. |
| UID registries | `agents.yml`, `inventory.yml`, `account-structure.yml` | Canonical subject existence. |
| RBAC registries | `RBAC_SCHEMA.md`, future structured RBAC config | Canonical tier/capability ceiling. |
| Org session registry | `MEMORY/STATE/org-sessions/` | Scoped role assignment source. |

Private keys remain outside `~/.claude` except temporary issuer context. Public metadata and DID docs may live under PAI config/state.

## Lifecycle

| State | Meaning | Allowed |
|---|---|---|
| `proposed` | UID/DID requested, not trusted | No auth; audit only. |
| `active` | Identity verified and credentialed | Normal auth/RBAC. |
| `suspended` | Temporarily blocked | Deny new sessions; allow read-only audit. |
| `rotating` | Old and new key overlap | Accept both keys inside overlap window. |
| `revoked` | Identity or key no longer trusted | Deny all except revocation audit lookup. |
| `expired` | Credential TTL ended | Deny until reissued. |

Rotation policy:

- Session credentials: 1 hour.
- Org role credentials: no longer than org session; usually 1 hour.
- Agent identity credentials: 90 days preferred; 365 days maximum during migration.
- Service identity credentials: 90-365 days based on service restart cost.
- SSH admin certs: 8 hours.
- TLS: 24 hours where ACME automation is live.

## Verification Algorithm

```text
verify_identity_proof(request, required_action, resource, context):
  1. Extract proof envelope: JWT, message proof, SSH cert, TLS client cert, or HMAC migration proof.
  2. Verify cryptographic signature against the declared verification method.
  3. Resolve `sub` / `from_did` to local DID document.
  4. Assert DID document UID exists in canonical registry.
  5. Assert DID status is active or rotating.
  6. Assert credential is not expired, revoked, or outside allowed clock skew.
  7. Assert credential_type is allowed for the action.
  8. Assert JWT `uid` matches DID document `uid`.
  9. Assert alias claims, if present, map to the same UID.
  10. Assert PoW is valid for inbox/Switchboard messages.
  11. Assert org role credential exists for org-scoped action.
  12. Pass canonical UID into RBAC authorize().
  13. Emit audit event with proof type, UID, DID, credential_id, action, resource, verdict.
```

Denial is mandatory when:

- A message claims `from` alias but lacks `from_uid` and verifiable proof.
- JWT `sub` is a loose string and no migration map binds it to a UID.
- A proof verifies cryptographically but resolves to a revoked DID/key.
- A credential's `uid`, `sub`, `kid`, or role assignment disagrees with registry state.
- Org role claims are presented as stable identity roles.

## RBAC Integration

Verifiable identity runs before RBAC:

```text
proof -> canonical UID/DID -> identity ceiling -> org role credential -> RBAC authorize()
```

RBAC must never authorize directly from:

- Display name.
- Inbox name.
- Model name.
- Claimed engine name.
- Unsigned `from`.
- HMAC-only shared secret once migration is complete.

RBAC audit event should include:

```json
{
  "actor_uid": "AGT-003",
  "actor_did": "did:pai:agt:003",
  "proof_type": "PAISessionCredential",
  "credential_id": "cred:pai:session:...",
  "identity_status": "active",
  "rbac_tier": "T1",
  "org_role_credential": "cred:pai:ora:001",
  "action": "task:update:owned",
  "resource": "TASK-001",
  "verdict": "allow"
}
```

## Migration Plan

### Phase 1: Structured Identity Registry

Create `config/verifiable-identities.yml` generated from:

- `agents.yml`
- `inventory.yml`
- `account-structure.yml`
- `RBAC_SCHEMA.md`
- `Tools/PKI/identities.json`

Minimum fields:

```yaml
identities:
  - uid: AGT-003
    did: did:pai:agt:003
    class: agent
    aliases: [PNK, opai, opencode]
    host_uid: SYS-001
    rbac_tier: T1
    status: active
    credential_subjects:
      legacy:
        - opai
      preferred:
        - did:pai:agt:003
    allowed_credentials:
      - PAIAgentIdentityCredential
      - PAISessionCredential
      - PAIOrgRoleCredential
      - PAIMessageProof
```

### Phase 2: JWT Claim Upgrade

Modify `Tools/PKI/issue.ts` so new JWTs include:

- `sub`: DID, not loose alias.
- `uid`: canonical UID.
- `kid`: key id in protected header.
- `credential_type`: identity/session/org-role.
- `rbac_tier`: copied from registry.
- `aud`: intended service where applicable.
- `scope`: still present for service compatibility.

Modify `Tools/PKI/verify.ts` so it can:

- Verify legacy tokens during migration.
- Return canonical UID/DID.
- Reject unknown subjects after migration deadline.
- Check revocation by `jti` and `kid`.

### Phase 3: Message Proof Upgrade

Modify inbox/Switchboard messages:

- Require `from_uid`, `from_did`, `to_uid`, `to_did`.
- Require PoW.
- Require Ed25519 proof or per-UID HMAC during transition.
- Treat `from` as alias only.

`InboxWatchdog.ts` should reject unsigned messages once migration ends; current `missing-sig-warn-pass` is acceptable only as a temporary migration state.

### Phase 4: Org Role Credentials

When TeamCreate/org-session creation assigns roles, issue short-lived `PAIOrgRoleCredential` documents for each assignment.

Org-session RBAC then verifies:

- Holder UID/DID.
- Org session ID.
- Structure and role.
- Grants/resource scope.
- Expiry.
- Separation-of-duty constraints.

### Phase 5: Audit and Revocation

Add verifier/audit tool:

```bash
bun ~/.claude/PAI/Tools/Identity/VerifyIdentity.ts --all
bun ~/.claude/PAI/Tools/Identity/VerifyIdentity.ts --uid AGT-003
bun ~/.claude/PAI/Tools/Identity/VerifyIdentity.ts --token ~/.config/PAI/pki/credentials/opai.jwt
```

Required outputs:

- UID/DID registry consistency.
- Credential validity.
- Expired/revoked keys.
- Legacy subject inventory.
- Missing DID documents.
- RBAC tier mismatch.
- Org-role credential mismatch.

## Verification Probes

Before enforcement:

| Probe | Expected |
|---|---|
| Parse `agents.yml` and `inventory.yml` | Every UID maps to exactly one DID. |
| Parse `Tools/PKI/identities.json` | Every subject maps to one UID or is flagged legacy-unmapped. |
| Verify public key exists | `~/.config/PAI/pki/public.jwk.json` readable by verifier. |
| Issue test DID JWT | `sub=did:pai:*`, `uid=*`, `credential_type=*`, `kid=*`. |
| Verify test DID JWT | Returns canonical UID/DID and scopes. |
| Verify revoked `jti` | Denied and logged. |
| Inbox message without proof | Denied after migration flag. |
| Inbox message with alias mismatch | Denied even if PoW valid. |
| Org role credential expired | Denied by org RBAC. |
| Audit log completeness | Every allow/deny logs UID, DID, credential id, action, verdict. |

## Anti-Model

Do not treat these as verifiable identities:

- A model saying "I am PAI Nova."
- A message field `from: PNX`.
- A path or filename under an inbox.
- A shared HMAC without a UID-bound key id.
- A PoW solution by itself.
- A JWT subject that is not bound to the UID registry.
- An Authentik group without a credential proof for the current session.

The invariant: **no authority from unverified names**. Names are labels; UID/DID plus current proof is identity.
