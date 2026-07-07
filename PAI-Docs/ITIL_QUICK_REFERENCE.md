# PAI ITIL Quick Reference
## Domain-Agnostic Service Management Cheat Sheet

**Version:** 1.0 | **Full Framework:** `ITIL_FRAMEWORK.md`

---

## Seven Guiding Principles (Apply to EVERY Decision)

| Principle | PAI Translation | Quick Check |
|-----------|-----------------|-------------|
| **Focus on Value** | Orient around principal value, not system elegance | "Does this create value for Duane?" |
| **Start Where You Are** | Build on existing PAI, don't rebuild | "Did I check for prior art?" (Preflight Gate E) |
| **Progress Iteratively** | Small increments + feedback | "Can I verify this incrementally?" |
| **Collaborate & Visibility** | Cross-engine sharing, transparent decisions | "Is this decision documented?" |
| **Think Systemically** | Consider whole-system impacts | "What else does this affect?" |
| **Keep It Simple** | Minimum steps, practical outcomes | "Is this complexity necessary?" |
| **Optimize & Automate** | Prove pattern, then automate | "Does this automate a validated workflow?" |

---

## Four Dimensions (Balance ALL Four)

| Dimension | PAI Components | Key Question |
|-----------|----------------|--------------|
| **People & Organization** | Principal, DA, RBAC personas, culture | "Does the principal have context to use this?" |
| **Information & Technology** | Engines (PNC/PNG/PNO/PNX/PNK), ICM, TELOS, Knowledge | "Is data accessible where needed?" |
| **Partners & Suppliers** | External APIs, MCP servers, integrations | "What if this external service fails?" |
| **Value Streams & Processes** | Algorithm phases, skills, hooks | "Can this workflow be simplified?" |

---

## Service Value Chain → Algorithm Phases

| SVC Activity | Algorithm Phase(s) | Purpose |
|--------------|--------------------|---------|
| **Plan** | OBSERVE, THINK, PLAN | Requirements, risk, strategy |
| **Improve** | REFLECT + Simplify | Learning capture, cleanup |
| **Engage** | (Cross-cutting) | Principal interaction, transparency |
| **Design & Transition** | PLAN | Skill dev, ISA scaffolding |
| **Obtain/Build** | BUILD, EXECUTE | Implementation |
| **Deliver & Support** | VERIFY, DELIVER | Verification, final checks |

---

## Key PAI Practices (From ITIL 34)

### Service Management

| Practice | PAI Implementation | When to Use |
|----------|-------------------|-------------|
| **Problem Detection** | Security hooks, HealthCheck, [Response Playbook](PROBLEM_RESPONSE_PLAYBOOK.md) | On errors, health degradation |
| **Root Cause Analysis** | RootCauseAnalysis skill, postmortems | Recurring problems, incidents |
| **Change Control** | ISA versioning, PhaseTransitionGate | Before modifying core components |
| **Knowledge Mgmt** | ICM, TELOS, reflection mining | Capturing/retrieving learnings |
| **Capability Registry** | 109 skills, 25+ agents | Discovering available capabilities |
| **State Tracking** | settings.json, ISA frontmatter | Managing configuration |
| **Quality Standards** | ISC floors, euphoric surprise threshold | Setting expectations |
| **Request Routing** | Mode detection, skill routing | Intake and triage |
| **Verification** | VERIFY phase, Evals, QATester | Before claiming complete |
| **Observability** | Hooks, HealthCheck, dashboard | Monitoring system health |

### General Management

| Practice | PAI Implementation | When to Use |
|----------|-------------------|-------------|
| **Continual Improvement** | REFLECT, Simplify, Loop, Optimize | Post-task, recurring work |
| **Security Governance** | Security hooks, audits, KeyRotation | Protecting credentials, blocking destructive ops |
| **Risk Management** | THINK premortem, WorldThreatModel | Before significant work, strategic decisions |
| **Strategy Setting** | TELOS missions, IDEAL_STATE | Quarterly planning, goal-setting |
| **Work Prioritization** | TELOS dependencies, effort tiers | Deciding what to work on |
| **System Design** | Architect agent, FirstPrinciples | Architecture decisions |
| **Metrics & Analytics** | ISC rates, euphoric surprise, logs | Measuring effectiveness |

### Technical Management

| Practice | PAI Implementation | When to Use |
|----------|-------------------|-------------|
| **Capability Release** | Cloudflare deployments, HealthCheck | Deploying new capabilities |
| **Resource Provisioning** | InfraOps, DBOps, systemd | Managing infrastructure |
| **Code Lifecycle** | Engineer agent, Algorithm BUILD | Software development |

---

## Governance Activities

| Activity | Purpose | PAI Implementation | Cadence |
|----------|---------|-------------------|----------|
| **DIRECT** | Set strategy & policy | AI Steering Rules, Algorithm doctrine, TELOS | Quarterly + ad-hoc |
| **MONITOR** | Oversee alignment | Hooks, dashboard, HealthCheck | Real-time (hooks), weekly (review) |
| **EVALUATE** | Review & update | REFLECT, feedback memories, BitterPillEngineering | Post-task, monthly, quarterly |

---

## Continual Improvement Loop

```
MEASURE → ANALYZE → IMPROVE → REPEAT

Measure:  ISC rates, euphoric surprise, errors, skill usage
Analyze:  Reflection mining, recurring patterns
Improve:  Simplify, BitterPillEngineering, feedback memories
Repeat:   Loop, Optimize, next Algorithm run
```

---

## Critical Success Factors & KPIs

| CSF | KPI | Target | Check |
|-----|-----|--------|-------|
| **Value Delivery** | Euphoric surprise | ≥8/10 | ISA rating |
| **Quality** | ISC pass rate | ≥95% | VERIFY results |
| **Learning** | Feedback memory rate | ≥1 per failed ISC | `MEMORY/Feedback/` |
| **Efficiency** | Ollama routing % | ≥60% for structured work | Inference.ts logs |
| **Reliability** | Uptime | ≥99.9% | HealthCheck |
| **Security** | Blocked ops | 100% block or override | DestructiveOpGuard logs |

---

## Daily Application Checklist

### Before Starting Work
- [ ] Apply "Focus on Value": Does this create principal value?
- [ ] Apply "Start Where You Are": Did I check for prior art?
- [ ] Check relevant dimension: People (context?), Tech (accessible?), Partners (fallback?), Process (simple?)

### During Work
- [ ] Apply "Progress Iteratively": Can I verify incrementally?
- [ ] Apply "Think Systemically": What else does this affect?
- [ ] Use appropriate practices (see tables above)

### After Work
- [ ] Apply "Optimize & Automate": Can proven patterns be automated?
- [ ] Run Continual Improvement loop: Measure, Analyze, Improve
- [ ] Capture learnings (REFLECT, feedback memories)

---

## When Things Go Wrong

| Symptom | ITIL Practice | PAI Tool |
|---------|---------------|----------|
| Error occurred | Problem Detection | Security hooks, HealthCheck |
| Error recurring | Root Cause Analysis | RootCauseAnalysis skill, postmortem |
| Risk identified | Risk Management | THINK premortem, RedTeam |
| Change needed | Change Control | ISA versioning, PhaseTransitionGate |
| Knowledge gap | Knowledge Management | Research, ICM recall |
| Quality issue | Service Validation | VERIFY phase, RealityChecker |
| Security concern | Security Governance | InfoSecRiskAssessment, SecurityTriage |

---

## Integration Map: ITIL ↔ PAI

| ITIL Concept | PAI Equivalent |
|--------------|----------------|
| Service Value System | Algorithm + hooks + skills ecosystem |
| Service Value Chain | OBSERVE→THINK→PLAN→BUILD→VERIFY→DELIVER→REFLECT |
| Guiding Principles | AI Steering Rules (system + user) |
| Four Dimensions | PAI architecture (People/Tech/Partners/Process) |
| Practices | Skills (109) + specialized agents |
| Governance | Direct (steering), Monitor (hooks), Evaluate (REFLECT) |
| Continual Improvement | REFLECT + Simplify + Loop + Optimize |
| Service Catalog | Skill registry + agent types |
| Configuration Management | settings.json + ISA frontmatter + work.json |
| Incident Management | Security hooks + error recovery |
| Problem Management | RootCauseAnalysis + feedback memories |
| Change Control | ISA versioning + PhaseTransitionGate |
| Knowledge Management | ICM + TELOS + Knowledge Archive |

---

**Pro Tip:** Print this page and keep it visible during PAI work. The principles and dimensions apply to EVERY decision.

**See Also:**
- `ITIL_FRAMEWORK.md` — Full framework documentation
- `AISTEERINGRULES.md` — System-wide behavioral rules
- `Algorithm/v5.7.3.md` — Algorithm doctrine
