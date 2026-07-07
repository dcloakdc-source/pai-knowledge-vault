# PAI RBAC Model for Org Structures

Source artifacts:
- `/home/duane/.claude/PAI/RBAC_SCHEMA.md`
- `/home/duane/.claude/PAI/IAM_ARCHITECTURE.md`
- `/home/duane/.claude/PAI/config/account-structure.yml`
- `/home/duane/.claude/PAI/config/agents.yml`
- `/home/duane/.claude/skills/Agents/Structures/`

## Purpose

PAI already has identity-tier RBAC for humans, engines, agents, systems, secrets, and memory. The new org structures add session-scoped coordination roles such as Guardian, Volunteer, CEO, L1 Service Desk, CISO, and Red Team.

These must not become global security roles. The RBAC model is therefore two-layered:

1. **Identity RBAC**: persistent trust tier and baseline capabilities for a UID.
2. **Org-Scoped RBAC**: temporary role grants inside a structure/team/session/resource boundary.

Effective access is the intersection of both layers, never their union.

```text
effective_permission =
  identity_baseline(UID)
  ∩ org_role_grant(structure, role)
  ∩ resource_scope(team/session/ticket/asset)
  ∩ active_guardrails(JIT, approval, severity, gate)
```

## Core Entities

| Entity | Key | Notes |
|---|---|---|
| Principal | `USR-001` | Duane; only principal accepts risk, spend, destructive ops, and authorization expansions. |
| Agent / Engine | `AGT-###` | Persistent actor identity from `agents.yml`. |
| System | `SYS-###` | Host/node identity from inventory. |
| Asset / Service | `AST-###` | Vaultwarden, Authentik, services, devices, protected applications. |
| Account | `ACC-###` | T0/T1/T2 sovereign account from `account-structure.yml`. |
| Org Session | `ORG-{timestamp}-{slug}` | Runtime instance of Family, Community, Business, IT Department, or InfoSec Org. |
| Org Role Assignment | `ORA-{id}` | Binds one UID to one org role in one org session. |
| Task / Ticket | `TASK-{id}` | Unit of work; may be owned by a scoped role. |
| Grant | `GRANT-{id}` | Optional explicit permission grant; JIT, approval, or exception. |

## Role Layers

### Layer 0: Sovereign Tier

Persistent, global tier from PAI RBAC:

| Tier | Members | Baseline |
|---|---|---|
| T0 Sovereign | `USR-001`, `AGT-001` | Full system control, RBAC admin, secret rotation, JIT issue/revoke. |
| T1 Privileged | `AGT-002`, `AGT-003`, `AGT-005` | Allowed production secrets, scoped memory write, agent tasking, JIT request/use. |
| T2 Standard | `AGT-004`, ephemeral agents | Public memory read, observe-only unless explicitly delegated. |

### Layer 1: Structure Lead Roles

These roles may coordinate within their org session but do not acquire global admin power:

| Structure | Lead Role | Coordination Rights |
|---|---|---|
| Family | Guardian | Assign chores, call Partner approval for irreversible actions, escalate to principal/IT. |
| Community | Moderator | Facilitate rough consensus, close decision, route security findings out. |
| Business | CEO | Own scope, assign deliverables, report to principal, submit to gates. |
| IT Department | IT Manager | Own ticket queue, route L1/L2/L3, report estate status. |
| InfoSec Org | CISO | Classify severity, decide mitigate-vs-monitor, report risk to principal. |

### Layer 2: Functional Org Roles

Functional roles are scoped to an org session and resource set:

| Role Class | Examples | Allowed Pattern |
|---|---|---|
| Producer | Apprentice, Volunteer, Engineer, SysAdmin, L3 Engineer | May modify assigned task resources only if identity tier permits write/exec. |
| Verifier | Partner, Watch, QA, Reviewer, Board, SecurityTriage | Read-first; may block closure but not perform producer changes. |
| Operator | Service Desk, SysAdmin, Network Engineer, DBA | May probe and execute runbook-scoped ops; destructive ops require explicit approval. |
| Security | SOC T1, Threat Intel, Red Team, IR Commander, GRC | Evidence-chain required; Red Team limited to authorized scope; remediation handed to IT. |
| Archivist | Elder, Librarian, Housekeeper | May summarize/write approved session artifacts; cannot expand access. |

### Layer 3: Universal Social Structures

Family, household, community, civic, tribal/clan, religious, educational, mutual aid, guild, movement, network, and emergency structures are coordination patterns. They map to the same RBAC primitives as business, IT, and InfoSec structures. They do not create new persistent trust tiers.

| Structure | Lead Role | Primary Function | Default RBAC Posture |
|---|---|---|---|
| Family | Guardian | Care, safety, schedule, family-task coordination. | Task coordination and approval request only. |
| Household | Steward / Housekeeper | Shared-space maintenance, routines, cleanup. | Owned cleanup tasks and session archive only. |
| Community | Moderator | Consensus, norms, decision closure. | Session coordination and decision routing. |
| Civic | Clerk / Advocate / Official | Procedure, representation, public-record discipline. | Record, route, and escalate; no privileged infra by title. |
| Tribal / Clan | Elder / Delegate | Continuity, dispute routing, cultural memory. | Read/search/write summary artifacts only. |
| Religious / Spiritual | Elder / Caretaker | Counsel, ritual, service, sacred memory. | Care-task and memory-summary scope only. |
| Educational | Teacher / Mentor | Assign learning work and assess progress. | Assign/update learning tasks; assessor gate separated. |
| Mutual Aid | Coordinator | Needs matching, distribution, volunteer routing. | Create/assign scoped tasks; no forced assignment. |
| Guild / Professional | Master / Reviewer | Craft standards, mentorship, certification. | Owned production plus independent review gate. |
| Movement | Organizer / Spokesperson | Mobilization, messaging, research. | Message/team coordination; publication requires gate. |
| Network | Hub / Broker | Connection, information routing, reciprocity. | Message/read routing; no write authority by default. |
| Emergency | Incident Commander | Temporary crisis coordination. | Active only under declared incident scope and expiry. |

### Layer 4: Role Archetypes

Every human-readable title should reduce to one or more archetypes before permissions are granted. The archetype decides the permission shape; the title remains display/context.

| Archetype | Common Titles | Allowed Pattern | Hard Limit |
|---|---|---|---|
| Principal | Duane, owner, sovereign sponsor | Accept risk, expand authorization, approve spend/destructive ops. | Only `USR-001` or explicit T0 artifact. |
| Coordinator | Guardian, Moderator, CEO, Product Owner, Platform Owner, Teacher, Organizer, Incident Commander, Agent Handler | Assign, route, close session decisions, request approvals. | Cannot exceed identity tier or bypass gates. |
| Producer | Apprentice, Volunteer, Engineer, Student, Responder, Toolsmith, Data Steward, SRE | Update owned tasks and write work evidence. | Cannot approve own work. |
| Verifier | Partner, Watch, QA, Reviewer, Assessor, Board, Access Reviewer, Model Evaluator, Design Reviewer | Read, challenge, block closure, certify outcome. | Cannot perform producer changes in same gate. |
| Operator | Service Desk, SysAdmin, DBA, Logistics Lead, Release Manager, Backup Operator, Change Manager | Execute runbook-scoped operations on assigned assets/tickets. | Destructive or undocumented actions require approval. |
| Security | CISO, SOC, Red Team, GRC, IR Commander, Security Architect, Vulnerability Manager, Forensics Lead, Privacy Steward | Triage, scan authorized scope, preserve evidence, respond. | Cannot accept risk unless T0. |
| Archivist | Elder, Librarian, Clerk, Scribe, Housekeeper, Memory Curator, Records Manager, Knowledge Reviewer | Preserve knowledge, summaries, decisions, audit trails. | Cannot expand access or mutate source facts. |
| Broker | Connector, Hub, Delegate, Advocate, Spokesperson, Customer Advocate | Route messages, introductions, requests, and opportunities. | Cannot bind others to obligations. |
| Caregiver | Parent, caretaker, counselor, support lead, Calendar Steward, Care Plan Coordinator, Document Clerk | Track care tasks, needs, safety checks. | No secret/infra/destructive rights by care title. |

### Responsibility Translation

Responsibilities translate to domain/action/scope permissions. Assign the smallest permission that lets the role complete the job.

| Responsibility | Permission Pattern | Typical Gate |
|---|---|---|
| Care / support | `task:update:owned`, `memory:write:summary` | Guardian or Caregiver review. |
| Provision / logistics | `task:create:team`, `task:assign:team` | Coordinator approval. |
| Governance / decision closure | `team:close:session`, `approval:approve:session` | Independent verifier or principal gate. |
| Culture / memory | `memory:write:knowledge`, `memory:archive:session` | Archivist plus coordinator close. |
| Education / mentorship | `task:assign:team`, `task:update:owned` | Assessor/reviewer separation. |
| Defense / safety | `security:triage:*`, `security:scan:authorized-scope` | Principal-approved scope for active testing. |
| Coordination | `agent:message:team`, `team:assign:session` | Session boundary and expiry. |
| Repair / conflict | `task:update:team`, `approval:deny:session` | Partner/Watch/Moderator review. |
| Succession / delegation | `task:assign:team`, `memory:write:knowledge` | T0 required for RBAC authority transfer. |

### Responsibility Bundles

Responsibilities are accountability labels, not direct permissions. They make it clear what a role owns while the grants/blocks still determine what the role can do.

| Bundle | Existing Archetype | Existing Role Fit | Permission Translation |
|---|---|---|---|
| Platform ownership | Coordinator | CEO, IT Manager, Platform Owner | `task:assign:team`, `team:assign:session`, `approval:request:*` |
| Reliability / SRE | Producer or Operator | Engineer, Engineering L3, Responder | `task:update:owned`, `infra:status:asset`, `infra:runbook:ticket` |
| Release / change coordination | Operator | IT Manager, SysAdmin L2, Release Manager, Change Manager | `infra:status:asset`, `infra:runbook:ticket`, `approval:request:destructive` |
| Backup / restore assurance | Operator or Verifier | DBA L2, Backup Operator, QA | `infra:status:database`, `task:update:ticket`, `task:close:gate` |
| Access review | Verifier or Security | Partner, QA, Access Reviewer, CISO | `memory:read:work`, `task:update:review`, `security:triage:session` |
| Finding lifecycle | Security | CISO, SOC T1, SecurityTriage, Vulnerability Manager | `security:triage:*`, `memory:write:evidence`, `task:update:ticket` |
| Evidence preservation / forensics | Security | IR Commander, Forensics Lead, Red Team | `memory:write:evidence`, `security:respond:session` |
| Privacy / data boundary | Security or Archivist | Privacy Steward, Records Manager, Librarian | `memory:read:work`, `memory:archive:session`, `approval:request:risk` |
| Memory curation | Archivist | Librarian, Memory Curator, Knowledge Reviewer | `memory:write:summary`, `memory:write:knowledge`, `memory:archive:session` |
| Records retention | Archivist | Clerk, Scribe, Records Manager | `memory:archive:session`, `memory:write:summary` |
| Agent work routing | Coordinator | Agent Handler, PM, IT Manager | `agent:message:team`, `task:assign:team` |
| Prompt / playbook maintenance | Producer or Archivist | Toolsmith, Prompt Librarian, Engineer | `memory:write:work`, `memory:write:summary` |
| UI / UX / visual design | Producer and Verifier | Designer, Artist, UIReviewer, Design Reviewer | `task:update:owned`, `memory:write:work`, `task:update:review`, `task:close:gate` |
| Model evaluation | Verifier | QA, Reviewer, Model Evaluator | `task:update:review`, `task:close:gate`, `memory:read:work` |
| Stakeholder / customer advocacy | Broker or Verifier | Spokesperson, Customer Advocate, Board | `agent:message:team`, `memory:read:work`, `approval:deny:session` |
| Personal operations | Caregiver or Archivist | Guardian, Calendar Steward, Document Clerk | `task:update:owned`, `memory:write:summary` |

### Archetype Engagement

Engagement decides which archetypes should participate before any org-session role assignment exists. It is advisory until an `ORG-*` session and scoped `ORA-*` assignments are created.

```text
task text + structured context signals + requested actions
  -> archetype engagement policy
  -> recommended archetypes and default structure
  -> coordinator creates proposed ORA assignments
  -> org RBAC authorizer accepts or denies scoped actions
```

Engagement policy path:

- Example only: `~/.claude/PAI/config/archetype-engagement.example.yml`
- Future live path: `~/.claude/PAI/config/archetype-engagement.yml`

Validation probe:

```bash
python3 ~/.claude/PAI/Tools/validate-archetype-engagement.py \
  ~/.claude/PAI/config/archetype-engagement.example.yml
```

Fixture probe:

```bash
python3 ~/.claude/PAI/Tools/validate-archetype-engagement.py --fixture-run \
  ~/.claude/PAI/config/archetype-engagement.example.yml
```

Classification probe:

```bash
python3 ~/.claude/PAI/Tools/validate-archetype-engagement.py --json \
  --classify "harden the FunctionsAPI identity route and add validator tests" \
  --context-signals code_change,touches_identity \
  ~/.claude/PAI/config/archetype-engagement.example.yml
```

The classifier is read-only. It does not grant permissions, create sessions, create `ORA-*` assignments, issue credentials, or write audit logs. It only emits a dry-run recommendation with trace evidence.

Candidate preferences may name preferred UIDs for an archetype, such as `AGT-008` as Cato Reviewer for Verifier/Security review or Forge Worker for Producer implementation. These are routing hints only. The org RBAC authorizer must still check identity tier, session assignment, scope, lifecycle, and guardrails before any permission is effective.

Engagement uses two signal classes:

| Signal Type | Source | Purpose |
|---|---|---|
| Keywords | User/task text such as `build`, `identity`, `backup`, `archive`, `notify`. | Fast conversational routing. |
| Context signals | Structured flags such as `touches_identity`, `destructive_action`, `memory_write`, `external_message`. | Reliable routing from tools, tickets, and future session creators. |

Mandatory pairings preserve gate discipline:

| Triggered Archetype | Added Archetype | Reason |
|---|---|---|
| Producer | Verifier | Producer cannot close its own gate. |
| Security | Verifier | Security-sensitive work needs independent review. |
| Operator | Verifier | Infra/runbook work needs an operational check. |
| Broker | Verifier | Publication or external communication needs a gate. |

Principal engagement is explicit. It is required when keywords, context, or requested actions imply risk acceptance, destructive operations, spend, global authority, secret rotation, RBAC admin, or sudo-session capability.

Design engagement is handled as capability routing, not a new authority class. UI, UX, art, frontend, visual, accessibility, responsive, theme, palette, and typography triggers engage Producer for creation/implementation and Verifier for UX review, accessibility review, visual regression, and closure gates.

### Role Lifecycle

Org-scoped authority must have a lifecycle. A role without lifecycle state is advisory only.

| State | Meaning | Enforcement |
|---|---|---|
| Proposed | Role requested but not active. | No grants. |
| Active | Role assigned inside an org session. | Grants usable within scope and tier ceiling. |
| Suspended | Role paused due to conflict, incident, expiry concern, or review. | Deny all mutating actions. |
| Escalated | Role has requested approval/JIT/incident authority. | Await required grant; no implied access. |
| Closed | Session or role completed. | Grants expired; audit retained. |
| Revoked | Assignment removed before normal close. | Deny and log future attempts. |

### Escalation Model

Escalation is explicit and evidence-bound:

1. Advisory roles may recommend but cannot grant.
2. Coordinators may request approval but cannot self-approve privileged action.
3. Verifiers may block closure but cannot mutate the deliverable being verified.
4. Operators may execute documented runbooks inside asset/ticket scope.
5. Security roles may classify severity and preserve evidence; risk acceptance stays T0.
6. Emergency roles expire automatically when the incident closes or the grant expires.

## Permission Namespace

Use domain/action/scope permissions. Existing permissions map cleanly into this form.

| Domain | Actions | Examples |
|---|---|---|
| `memory` | `read`, `write`, `search`, `archive` | `memory:write:work`, `memory:read:public` |
| `agent` | `spawn`, `task`, `message`, `revoke`, `observe` | `agent:spawn:team`, `agent:task:team` |
| `team` | `create`, `delete`, `assign`, `close` | `team:assign:session` |
| `task` | `create`, `claim`, `assign`, `update`, `close` | `task:claim:team`, `task:close:owned` |
| `secret` | `listed`, `jit`, `rotate`, `deny` | `secret:jit:single-use` |
| `infra` | `status`, `runbook`, `manage`, `destructive` | `infra:runbook:asset`, `infra:destructive:approved` |
| `security` | `triage`, `scan`, `respond`, `audit`, `accept-risk` | `security:scan:authorized-scope` |
| `approval` | `request`, `approve`, `deny` | `approval:request:destructive` |
| `budget` | `read`, `limit`, `spend` | `budget:read:session` |
| `rbac` | `read`, `admin` | `rbac:admin:global` |
| `system` | `status`, `restart` | `system:restart:approved` |

Scope suffixes:

| Scope | Meaning |
|---|---|
| `global` | All PAI resources; T0 only by default. |
| `session` | Current org session/team only. |
| `team` | Current team membership and task list. |
| `owned` | Resources explicitly assigned to the actor. |
| `asset:{AST/SYS}` | Named service, host, or asset. |
| `ticket:{id}` | One IT/security ticket. |
| `authorized-scope` | Explicit principal-approved security test scope. |

## Structure Role Grants

### Family

| Role | Grants | Blocks |
|---|---|---|
| Guardian | `team:assign:session`, `task:create:team`, `agent:message:team`, `approval:request:destructive` | Cannot bypass Partner/principal approval for irreversible actions. |
| Partner | `approval:approve:session`, `task:update:team`, `memory:read:work` | Cannot execute destructive action directly. |
| Elder | `memory:read:work`, `memory:search:work`, `memory:write:summary` | No secret, infra, or destructive rights. |
| Apprentice | `task:update:owned`, `memory:write:work` | Own task only; no approval authority. |
| Housekeeper | `task:close:owned`, `memory:archive:session` | Cleanup only; no deletion outside session temp artifacts. |

### Community

| Role | Grants | Blocks |
|---|---|---|
| Moderator | `team:close:session`, `task:update:team`, `agent:message:team` | Cannot assign owned work except to break deadlock. |
| Organizer | `task:create:team`, `task:update:team` | Creates unowned work; cannot force volunteers. |
| Volunteer | `task:claim:team`, `task:update:owned`, `memory:write:work` | Cannot write final knowledge artifacts directly. |
| Librarian | `memory:write:knowledge`, `memory:archive:session` | Writes curated findings only after Moderator close. |
| Watch | `memory:read:work`, `task:update:team` | Read/cross-check only; cannot own findings. |

### Household

| Role | Grants | Blocks |
|---|---|---|
| Steward | `task:create:team`, `task:assign:team`, `memory:write:summary` | Cannot approve destructive household/infra changes. |
| Housemate | `task:claim:team`, `task:update:owned` | Own tasks only; no assignment authority. |
| Maintainer | `infra:status:asset`, `infra:runbook:ticket`, `task:update:ticket` | Runbook-scoped only; repairs that change infra escalate to IT. |
| Guest | `memory:read:public`, `task:update:owned` | Temporary, narrow task scope only. |

### Education / Guild

| Role | Grants | Blocks |
|---|---|---|
| Teacher / Mentor | `task:create:team`, `task:assign:team`, `memory:write:work` | Cannot certify own curriculum or final assessment. |
| Student / Apprentice | `task:update:owned`, `memory:write:work` | Cannot close assessment gate. |
| Assessor / Master Reviewer | `task:close:gate`, `task:update:review`, `memory:read:work` | Cannot rewrite submitted work while acting as gate. |
| Scribe | `memory:write:summary`, `memory:archive:session` | Cannot alter source records. |

### Mutual Aid / Movement / Network

| Role | Grants | Blocks |
|---|---|---|
| Coordinator | `task:create:team`, `task:assign:team`, `agent:message:team` | Cannot force volunteer ownership or expand scope. |
| Volunteer | `task:claim:team`, `task:update:owned`, `memory:write:work` | Cannot publish final artifacts directly. |
| Spokesperson | `agent:message:team`, `memory:read:work`, `memory:write:summary` | External publication requires gate approval. |
| Broker / Hub | `agent:message:team`, `memory:read:public` | Cannot bind principals, teams, or assets to obligations. |
| Researcher | `memory:write:work`, `task:update:owned` | External content remains untrusted until reviewed. |

### Emergency / Crisis

| Role | Grants | Blocks |
|---|---|---|
| Incident Commander | `task:assign:incident`, `security:respond:sev1-2`, `approval:request:destructive` | Active only during declared incident; cannot accept risk. |
| Responder | `task:update:owned`, `infra:runbook:ticket`, `memory:write:evidence` | No undocumented or destructive action without approval. |
| Communications Lead | `agent:message:team`, `memory:write:summary` | Cannot issue technical authority or risk acceptance. |
| Logistics Lead | `task:create:team`, `task:assign:team`, `budget:read:session` | Spend requires principal approval. |

### Business

| Role | Grants | Blocks |
|---|---|---|
| CEO | `team:assign:session`, `task:assign:team`, `approval:request:*`, `budget:read:session` | Cannot overrule failed QA/Board gates. |
| COO | `task:assign:team`, `task:update:team`, `agent:message:team` | No scope changes without CEO. |
| PM | `task:create:team`, `task:update:requirements`, `memory:write:work` | Read-only regarding implementation. |
| Engineer | `task:update:owned`, `memory:write:work`, `infra:status:owned` | No production deploy/destructive ops without approval. |
| QA | `task:close:gate`, `memory:read:work`, `infra:status:owned` | Blocks closure; does not implement fixes. |
| Reviewer | `task:update:review`, `memory:read:work` | Read-only on diffs. |
| Board | `task:close:gate`, `approval:deny:session` | Final gate; cannot directly modify deliverables. |

### IT Department

| Role | Grants | Blocks |
|---|---|---|
| IT Manager | `task:assign:team`, `team:close:session`, `infra:status:asset`, `approval:request:destructive` | Cannot accept security risk. |
| Service Desk L1 | `infra:status:asset`, `infra:runbook:ticket`, `task:update:ticket` | Documented runbooks only; undocumented work escalates. |
| SysAdmin L2 | `infra:status:asset`, `infra:manage:asset`, `memory:write:runbook` | Destructive ops require principal approval. |
| Network Eng L2 | `infra:status:network`, `infra:manage:network`, `memory:write:runbook` | Route/firewall destructive changes require approval. |
| DBA L2 | `infra:status:database`, `infra:manage:database`, `memory:write:runbook` | Drop/truncate/migrate require approval and backup probe. |
| Engineering L3 | `task:update:owned`, `memory:write:work`, `infra:manage:approved-asset` | Architecture changes become Business project scope. |

### InfoSec Org

| Role | Grants | Blocks |
|---|---|---|
| CISO | `security:triage:session`, `security:respond:session`, `approval:request:risk`, `task:assign:team` | Cannot accept risk for Duane. |
| SOC T1 | `security:triage:signal`, `memory:write:evidence`, `task:update:ticket` | No remediation execution. |
| SecurityTriage | `security:triage:finding`, `memory:write:evidence` | Skill invocation only; no direct infra modification. |
| Threat Intel | `memory:write:intel`, `security:triage:context` | External content treated as untrusted. |
| Red Team | `security:scan:authorized-scope`, `memory:write:evidence` | No out-of-scope testing; no patching. |
| IR Commander | `security:respond:sev1-2`, `task:assign:incident` | Only active for SEV1/SEV2 or explicit activation. |
| GRC | `security:audit:policy`, `memory:write:evidence` | Audits coverage; does not implement controls. |

## Inheritance and Deny Rules

1. **Identity ceiling**: an org role cannot grant more than the actor's persistent tier permits.
2. **Session boundary**: org grants expire when the team/session closes.
3. **Resource binding**: producer roles operate only on owned tasks/assets.
4. **Gate separation**: producer and verifier roles cannot close the same deliverable without an independent gate role.
5. **Red/blue separation**: Red Team cannot remediate; blue/IT cannot retroactively authorize a red probe.
6. **Principal-only decisions**: risk acceptance, spend, authorization expansion, and destructive operations require Duane or an explicit T0 approval artifact.
7. **Deny beats allow**: `deny` from guardrail, missing approval, expired JIT, severity stop, or out-of-scope asset overrides any role grant.
8. **No global role leakage**: Guardian/CEO/CISO/IT Manager/Moderator are org-session roles, not Authentik groups.

## Data Schema

Minimum YAML registry shape:

```yaml
org_sessions:
  - id: ORG-20260614-rbac-example
    structure: business
    team_name: venture-example
    created_by: AGT-001
    status: active
    lifecycle_state: active
    resource_scope:
      memory_paths:
        - MEMORY/WORK/20260614-rbac-example/
      assets: []
      repos: []
    assignments:
      - id: ORA-001
        uid: AGT-003
        role: Engineer
        structure: business
        team_name: venture-example
        task_scope:
          - TASK-001
        grants:
          - task:update:owned
          - memory:write:work
        lifecycle_state: active
        expires_at: 2026-06-14T23:59:59Z
    approvals:
      - id: GRANT-001
        type: destructive-op
        requested_by: AGT-001
        approved_by: USR-001
        scope: asset:SYS-001
        expires_at: 2026-06-14T18:00:00Z
```

Recommended storage:

- Canonical definitions: `~/.claude/PAI/config/org-rbac.yml`
- Runtime sessions root: `MEMORY/STATE/org-sessions/`
- Active runtime sessions: `MEMORY/STATE/org-sessions/active/{ORG_ID}.yml`
- Closed session archive: `MEMORY/STATE/org-sessions/closed/{ORG_ID}.yml`
- Revoked/security-stopped sessions: `MEMORY/STATE/org-sessions/revoked/{ORG_ID}.yml`
- Non-authoritative examples: `MEMORY/STATE/org-sessions/examples/ORG-EXAMPLE-{structure}.yml`
- Audit log: `MEMORY/SECURITY/org-rbac-audit.jsonl`

Files must not be stored directly under `MEMORY/STATE/org-sessions/` except `README.md` and directory placeholders. Moving a session file between `active/`, `closed/`, and `revoked/` is a lifecycle state transition; use `Tools/org-session-manager.py` for normal lifecycle changes.

## Enforcement Contract

The markdown model is the human source. Runtime enforcement should compile from a structured registry with four tables:

1. `structures`: known structure IDs, allowed roles, default lifecycle policy.
2. `archetypes`: reusable permission and hard-limit templates.
3. `role_bindings`: structure-specific title to archetype mappings.
4. `constraints`: global deny, separation-of-duty, lifecycle, approval, and expiry rules.

Example contract path:

- Example only: `~/.claude/PAI/config/org-rbac.example.yml`
- Future live path: `~/.claude/PAI/config/org-rbac.yml`

Validation probe:

```bash
python3 ~/.claude/PAI/Tools/validate-org-rbac.py ~/.claude/PAI/config/org-rbac.example.yml
```

Fixture probe:

```bash
python3 ~/.claude/PAI/Tools/validate-org-rbac.py --fixture-run ~/.claude/PAI/config/org-rbac.example.yml
```

Dry-run authorization probe:

```bash
python3 ~/.claude/PAI/Tools/validate-org-rbac.py --authorize \
  --actor-uid AGT-003 \
  --structure business \
  --role Engineer \
  --action task:update:owned \
  --resource TASK-001 \
  ~/.claude/PAI/config/org-rbac.example.yml
```

Session dry-run authorization probe:

```bash
python3 ~/.claude/PAI/Tools/validate-org-rbac.py --authorize \
  --actor-uid AGT-003 \
  --action task:update:owned \
  --resource TASK-001 \
  --session-file ~/.claude/PAI/MEMORY/STATE/org-sessions/examples/ORG-EXAMPLE-business.yml \
  ~/.claude/PAI/config/org-rbac.example.yml
```

Audit-preview probe:

```bash
python3 ~/.claude/PAI/Tools/validate-org-rbac.py --json --audit-preview --authorize \
  --actor-uid AGT-003 \
  --action task:update:owned \
  --resource TASK-001 \
  --session-file ~/.claude/PAI/MEMORY/STATE/org-sessions/examples/ORG-EXAMPLE-business.yml \
  ~/.claude/PAI/config/org-rbac.example.yml
```

Self-test gate:

```bash
python3 ~/.claude/PAI/Tools/validate-org-rbac.py --self-test ~/.claude/PAI/config/org-rbac.example.yml
```

Advisory agent/job route:

```bash
python3 ~/.claude/PAI/Tools/agent-job-routing.py \
  --task "Fix failing GitHub Actions CI for a Cloudflare Worker PR" \
  --resource TASK-001 \
  --json
```

The router recommends structure, role, engine, skills, an RBAC request draft, and `org-session-manager.py` command drafts. It does not create sessions, assign authority, or bypass validation.

Switchboard movement preflight:

```bash
python3 ~/.claude/PAI/Tools/switchboard-rbac-router.py \
  --actor-uid AGT-001 \
  --target execute \
  --resource PLAN-001 \
  --session ORG-YYYYMMDD-HHMMSS-session \
  --role Moderator \
  --json
```

Switchboard dispatches and Kanban moves must stop on a `deny` verdict. The preflight checks active org-session state through `validate-org-rbac.py`; it does not create sessions or grant authority.

Live enforcement point:

```bash
python3 ~/.claude/PAI/Tools/org-rbac-live-enforcer.py \
  --actor-uid AGT-003 \
  --action task:update:owned \
  --resource TASK-001 \
  --session ORG-YYYYMMDD-HHMMSS-session \
  --role Engineer \
  --json
```

`org-rbac-live-enforcer.py` is the runtime guard boundary for org-scoped actions. It resolves the session, calls `validate-org-rbac.py`, returns an effective allow/deny verdict, and appends `MEMORY/SECURITY/org-rbac-live-enforcement.jsonl`. Principal/T0 override is explicit and audited:

```bash
python3 ~/.claude/PAI/Tools/org-rbac-live-enforcer.py \
  --actor-uid AGT-003 \
  --action task:close:gate \
  --resource TASK-001 \
  --session ORG-YYYYMMDD-HHMMSS-session \
  --role Engineer \
  --override \
  --override-by USR-001 \
  --override-reason "principal accepted closure risk" \
  --json
```

Overrides change only the effective verdict. The validator verdict and reason remain in the audit record.

JSON request probe:

```bash
python3 ~/.claude/PAI/Tools/validate-org-rbac.py \
  --request-json ~/.claude/PAI/config/examples/org-rbac-request-allow.json \
  ~/.claude/PAI/config/org-rbac.example.yml
```

Stdin request probe:

```bash
cat ~/.claude/PAI/config/examples/org-rbac-request-allow.json | \
  python3 ~/.claude/PAI/Tools/validate-org-rbac.py \
    --request-json - \
    ~/.claude/PAI/config/org-rbac.example.yml
```

The validator and dry-run authorizer are read-only. They do not issue credentials, mutate org sessions, write audit events, or promote the example contract/session to live enforcement. `--audit-preview` emits the future audit event shape in stdout only; it must not append to `MEMORY/SECURITY/org-rbac-audit.jsonl`. A dry-run `allow` is diagnostic only; live enforcement still requires guardrail integration.

### Canonical Keys

| Key | Required | Meaning |
|---|---|---|
| `schema_version` | yes | Contract version; incompatible changes increment major version. |
| `structures[].id` | yes | Stable lowercase structure ID such as `family`, `business`, `emergency`. |
| `structures[].roles[].title` | yes | Human-readable title, not an Authentik group. |
| `structures[].roles[].archetype` | yes | One of the registered archetypes. |
| `structures[].roles[].grants` | yes | Allow-list of domain/action/scope permissions. |
| `structures[].roles[].blocks` | yes | Explicit denies or non-authorities for that title. |
| `structures[].roles[].responsibilities` | no | Accountability labels for planning and audit; not direct permission grants. |
| `structures[].roles[].requires` | no | Required gate, approval, declared incident, or scope proof. |
| `structures[].roles[].max_lifecycle` | no | Highest lifecycle state this role can enter without T0. |
| `constraints[].id` | yes | Stable constraint ID for audit and test fixtures. |
| `constraints[].effect` | yes | `deny`, `require_approval`, `require_separation`, or `require_scope`. |

### Grant Compilation

Compile grants in this order:

```text
compiled_role(actor, assignment) =
  identity_tier_ceiling(actor.uid)
  ∩ archetype_template(assignment.archetype)
  ∩ structure_role_grants(assignment.structure, assignment.role)
  ∩ assignment_resource_scope(assignment)
  ∩ active_explicit_grants(assignment)
```

The compiler must reject any role binding that:

1. References an unknown archetype.
2. Uses a permission outside the known namespace.
3. Grants `global` scope to non-T0 identities.
4. Grants `secret:*`, `rbac:admin`, `security:accept-risk`, `budget:spend`, or `infra:destructive` without explicit T0 approval.
5. Has no `blocks` entry for authority-sensitive roles.
6. Omits expiry for emergency, security, JIT, or external participant roles.

### Deny Precedence

Deny rules evaluate before allows. The first matching deny should be logged with its constraint ID.

| Order | Deny Source | Example |
|---|---|---|
| 1 | Identity revoked or inactive | UID disabled, DID revoked, expired session credential. |
| 2 | Lifecycle not active | Proposed, suspended, closed, or revoked assignment. |
| 3 | Scope mismatch | Actor assigned to `TASK-001` attempts `TASK-002`. |
| 4 | Missing principal approval | Destructive, spend, risk acceptance, authority expansion. |
| 5 | Separation-of-duty violation | Producer tries to close own QA gate. |
| 6 | Red/blue conflict | Red Team tries to remediate finding it produced. |
| 7 | Expired JIT or explicit grant | Grant TTL elapsed or single-use token consumed. |
| 8 | Guardrail or severity stop | Safety hook, policy deny, SEV escalation freeze. |

### Constraint Types

Represent guardrails as data, not prose:

| Constraint | Required Fields | Example |
|---|---|---|
| `principal_only` | `actions`, `approver_uid` | `infra:destructive:*` requires `USR-001`. |
| `separation_of_duty` | `producer_archetypes`, `verifier_archetypes`, `same_resource` | Producer cannot close same deliverable. |
| `red_blue_separation` | `red_roles`, `blue_actions` | Red Team cannot remediate. |
| `lifecycle_gate` | `allowed_states`, `mutating_actions` | Mutations require `active`. |
| `scope_required` | `actions`, `scope_fields` | Security scans need `authorized_scope`. |
| `expiry_required` | `roles`, `max_ttl_minutes` | Emergency roles expire. |
| `publication_gate` | `roles`, `approver_archetypes` | Spokesperson drafts; verifier publishes. |

### Lifecycle Transitions

Lifecycle transitions are also authorization events:

| From | To | Required Authority |
|---|---|---|
| none | proposed | Coordinator or T0. |
| proposed | active | Coordinator plus identity-tier eligibility. |
| active | suspended | Coordinator, Verifier, Security, or T0. |
| suspended | active | Original coordinator plus verifier or T0. |
| active | escalated | Explicit approval/JIT/incident request. |
| escalated | active | Approval resolved or escalation revoked. |
| active | closed | Coordinator or session close gate. |
| any | revoked | T0, security stop, or expired identity credential. |

### Authorization Fixtures

Minimum fixture cases for an implementation test suite:

| Case | Expected |
|---|---|
| `business.engineer.update_owned_task` | allow |
| `business.engineer.close_qa_gate_same_task` | deny: separation_of_duty |
| `family.guardian.assign_task` | allow |
| `family.guardian.execute_destructive_infra` | deny: principal_only |
| `community.volunteer.write_final_knowledge` | deny: publication_or_archivist_gate |
| `education.student.close_assessment` | deny: separation_of_duty |
| `movement.spokesperson.publish_without_gate` | deny: publication_gate |
| `emergency.commander.assign_after_expiry` | deny: expired_assignment |
| `infosec.red_team.scan_without_scope` | deny: missing_authorized_scope |
| `infosec.red_team.remediate_own_finding` | deny: red_blue_separation |
| `it.l1.undocumented_destructive_op` | deny: principal_only |
| `t2.external.secret_jit_request` | deny: identity_ceiling |

## Enforcement Algorithm

```text
authorize(actor_uid, action, resource, context):
  1. Load actor from agents.yml / identity registry.
  2. Load identity tier from RBAC_SCHEMA.md-backed config.
  3. Load active org session and role assignments.
  4. Reject if no active assignment for org-scoped action.
  5. Reject if assignment lifecycle is not active.
  6. Normalize title to role archetype and grants.
  7. Reject if action exceeds identity baseline.
  8. Reject if action is outside assignment resource scope.
  9. Reject if a guardrail applies and required approval/JIT/SEV is absent.
  10. Reject if separation-of-duty rule is violated.
  11. Emit audit event with UID, session, role, archetype, action, resource, verdict.
  12. Permit.
```

Audit event:

```json
{
  "ts": "2026-06-14T16:31:27Z",
  "actor_uid": "AGT-003",
  "identity_tier": "T1",
  "org_session": "ORG-20260614-rbac-example",
  "structure": "business",
  "org_role": "Engineer",
  "role_archetype": "Producer",
  "action": "task:update:owned",
  "resource": "TASK-001",
  "verdict": "allow",
  "reason": "identity_baseline_and_org_scope_match"
}
```

## Migration Path

1. Convert the five structure YAML contracts into a generated `org-rbac.yml` role-grant registry.
2. Extend team/session creation to write `MEMORY/STATE/org-sessions/active/{ORG_ID}.yml`.
3. Add an authorization helper used by TeamCreate, TaskCreate/TaskUpdate, SendMessage, Skill invocation, and shell/secret guardrails.
4. Add role-archetype normalization so titles map to Coordinator, Producer, Verifier, Operator, Security, Archivist, Broker, or Caregiver before grants are evaluated.
5. Add lifecycle enforcement for proposed, active, suspended, escalated, closed, and revoked assignments.
6. Add separation-of-duty checks for Business gates, Education/Guild assessment gates, and InfoSec red/blue boundaries.
7. Add ticket/asset scoped checks for IT Department operations and Emergency response.
8. Add an audit command that verifies every org role has grants, blocks, scope, lifecycle, and expiry.

## Verification Probes

Required probes before treating this as enforced:

| Probe | Expected |
|---|---|
| Parse all five structure contracts | Every role has `kind`, `binding`, `required`, and valid structure ID. |
| Generate `org-rbac.yml` | All roles appear exactly once per structure. |
| Validate `org-rbac.example.yml` | `validate-org-rbac.py` exits 0 with zero errors and expected fixture verdicts. |
| Dry-run allow decision | Business Engineer `task:update:owned` returns `allow:allowed` with compiled grants. |
| Dry-run deny decision | Family Guardian `infra:destructive:asset` returns `deny:principal_only_authority`. |
| Session dry-run allow decision | Example session assignment resolves AGT-003 as Engineer for `TASK-001`. |
| Session dry-run scope deny | Example session assignment denies AGT-003 access to unassigned `TASK-002`. |
| Audit-preview dry-run | `--audit-preview` emits `dry_run: true`, verdict, reason, actor, session, role, action, resource, and trace. |
| Self-test gate | `--self-test` passes validation, fixtures, session allow/deny, and audit-preview shape checks. |
| JSON request interface | `--request-json` accepts file/stdin request objects and returns a JSON decision. |
| Spawn sample Business session | Engineer can update owned task; cannot close QA gate. |
| Spawn sample Community session | Volunteer can claim unowned task; cannot write final Knowledge artifact. |
| Spawn sample Family/Household session | Guardian/Steward can assign care or cleanup tasks; cannot execute destructive infra changes. |
| Spawn sample Education/Guild session | Student can update owned work; Assessor can close gate; same actor cannot do both for one deliverable. |
| Spawn sample Movement/Network session | Spokesperson can draft summary; publication denied without gate approval. |
| Spawn sample Emergency session | Incident Commander can assign incident tasks only while lifecycle is active and incident scope is declared. |
| Spawn sample IT session | L1 can runbook-resolve; cannot execute undocumented destructive op. |
| Spawn sample InfoSec session | Red Team scan denied without `authorized-scope`; remediation denied. |
| Expiry check | Role grant denied after session close or `expires_at`. |
| Lifecycle check | Proposed, suspended, closed, and revoked role assignments deny mutating actions. |
| Audit check | Every allow/deny writes UID, role, action, resource, verdict. |

## Anti-Model

Do not create Authentik groups named `pai-ceo`, `pai-ciso`, `pai-guardian`, `pai-volunteer`, or similar. Those are coordination roles, not identity-trust roles. Authentik should continue to hold stable IAM groups such as `pai-admin`, `pai-agent-privileged`, and `pai-agent-standard`; org roles belong in session state and audit logs.
