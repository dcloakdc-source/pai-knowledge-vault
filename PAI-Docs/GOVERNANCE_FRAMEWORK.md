# PAI Governance Framework
## Policies, Standards, and Decision Authority

**Version:** 1.0  
**Created:** 2026-06-21  
**Purpose:** Unified governance policies for PAI infrastructure and operations  
**Related:** `ITIL_FRAMEWORK.md`, `SECURITY_STANDARDS.md`, `GOVERNANCE_MODEL.md`

---

## Foundation: Seven Governing Principles

These principles (from ITIL v4) govern ALL PAI decisions and operations:

### 1. Focus on Value
**Policy:** Every PAI capability, process, and decision must create measurable value for the principal or advance TELOS missions (M1/M2/M3).

**Rationale:** PAI exists to serve Duane's missions, not to be an end in itself.

**Enforcement:** 
- Algorithm OBSERVE phase asks: "Does this create principal value?"
- ISC criteria must trace to value delivered
- Alignment tracking shows % time on mission-aligned work

**Exceptions:** Infrastructure work that enables future value (documented in ISA `## Decisions`)

---

### 2. Start Where You Are
**Policy:** Leverage existing PAI assets before building new ones. Assess current state before proposing changes.

**Rationale:** Prevents reinventing existing capabilities, reduces waste.

**Enforcement:**
- Algorithm Gate E (Existence Probe) - mandatory prior-art check
- Skills invoke existing tools before building new
- Migration/intake preserves prior work

**Exceptions:** When existing solution is fundamentally incompatible with new requirement (document why)

---

### 3. Progress Iteratively with Feedback
**Policy:** Break work into manageable increments. Use verification feedback to shape next iteration.

**Rationale:** Reduces risk, enables course correction, prevents over-investment in wrong direction.

**Enforcement:**
- Algorithm phases (OBSERVE→VERIFY→REFLECT) enforce iteration
- ISC verification provides feedback per criterion
- Reflection mining captures lessons for next iteration

**Exceptions:** None - iteration is always appropriate

---

### 4. Collaborate and Promote Visibility
**Policy:** Make decisions, reasoning, and system state visible. Enable transparency across engines and over time.

**Rationale:** Trust requires understanding. Debugging requires visibility.

**Enforcement:**
- ISA `## Decisions` section documents reasoning
- Hooks log actions to STATE.claude/
- Cross-engine work uses ICM (shared memory)
- Public daemon profile (selected work only)

**Exceptions:** Sensitive data (credentials, personal info) - use encryption, not visibility

---

### 5. Think and Work Systemically
**Policy:** Consider interdependencies. Optimize for whole-system outcomes, not local efficiency.

**Rationale:** Local optimizations often create global problems.

**Enforcement:**
- SystemsThinking skill for structural analysis
- Algorithm considers integration points
- Hooks enforce cross-cutting concerns (security, honesty)
- ApertureOscillation surfaces tactical vs strategic tensions

**Exceptions:** None - systemic thinking always applies

---

### 6. Keep It Simple and Practical
**Policy:** Minimize complexity. Choose simple solutions over clever ones. Pragmatic over perfect.

**Rationale:** Complexity is a liability. Simple systems are maintainable, understandable, debuggable.

**Enforcement:**
- Simplify skill post-implementation cleanup
- BitterPillEngineering audits for over-prompting
- "No ceremony tax" rule - don't let process eat budget

**Exceptions:** Irreducible complexity (document why simpler won't work)

---

### 7. Optimize and Automate
**Policy:** Automate repetitive work. Optimize high-frequency paths. But prove value manually first.

**Rationale:** Automation pays off over time. But premature automation wastes effort.

**Enforcement:**
- Hooks automate proven patterns (security checks, phase tracking)
- Systemd timers automate scheduled work
- Skills encapsulate reusable patterns
- Manual→Automated progression documented

**Exceptions:** One-time work (don't automate what runs once)

---

## Policy Categories

### 1. SECURITY POLICIES

**1.1 Secret Management**
- **Policy:** Secrets never committed to git, never in logs, rotated regularly
- **Enforcement:** OutputSecretsScanner hook, DestructiveOpGuard, KeyRotation skill
- **Standards:** See `SECURITY_STANDARDS.md` for details

**1.2 Access Control**
- **Policy:** Least privilege, explicit allow (deny by default)
- **Enforcement:** RBAC framework (`org-rbac.example.yml`), permissions in `settings.json`
- **Standards:** Zero Trust Architecture principles

**1.3 Data Protection**
- **Policy:** Sensitive data encrypted at rest and in transit
- **Enforcement:** LUKS encryption, Tailscale VPN, secret stores
- **Standards:** Encryption for PII, credentials, API keys

**1.4 Security Monitoring**
- **Policy:** Security events logged, reviewed, acted upon
- **Enforcement:** Problem Detection (4/5), security hooks, ITIL monthly review
- **Standards:** NIST CSF Detect function

---

### 2. PRIVACY POLICIES

**2.1 Data Minimization**
- **Policy:** Collect only necessary data, delete when no longer needed
- **Enforcement:** Manual enforcement (no auto-collection without explicit decision)
- **Standards:** GDPR-inspired (though personal use, not regulated)

**2.2 Consent & Agency**
- **Policy:** Others' data requires informed consent (especially kids)
- **Enforcement:** Family coordination respects kids' boundaries
- **Standards:** Age-appropriate consent, right to be forgotten

**2.3 Data Sovereignty**
- **Policy:** Duane's data stays under Duane's control (prefer local over cloud)
- **Enforcement:** ICM on pai-primary, Ollama local inference, encrypted backups
- **Standards:** Self-hosted where feasible, encrypted when cloud necessary

---

### 3. QUALITY POLICIES

**3.1 Verification Requirements**
- **Policy:** Every ISC must be verifiable via tool probe (not subjective judgment)
- **Enforcement:** Algorithm ISC quality system, granularity test
- **Standards:** ISC pass rate ≥95% (CSF metric)

**3.2 Euphoric Surprise Threshold**
- **Policy:** Significant work must achieve euphoric surprise ≥8/10
- **Enforcement:** Post-delivery reflection captures score
- **Standards:** E2+ effort targets euphoric surprise

**3.3 Code Quality**
- **Policy:** Code must be readable, maintainable, tested
- **Enforcement:** Simplify skill, biome linting, Algorithm verification phase
- **Standards:** TypeScript, Bun, functional patterns

---

### 4. OPERATIONS POLICIES

**4.1 Change Management**
- **Policy:** Significant changes require proposal → review → approval → deploy
- **Enforcement:** ISA workflow for multi-step work, git commits for code
- **Standards:** See `CORE_PROCESSES.md` - Change Management

**4.2 Incident Response**
- **Policy:** Security incidents require detection → triage → response → RCA
- **Enforcement:** Problem Detection (4/5), RCA (4/5), postmortem template
- **Standards:** See `PROBLEM_RESPONSE_PLAYBOOK.md`

**4.3 Backup & Recovery**
- **Policy:** Critical data backed up, backups tested via restore
- **Enforcement:** Systemd backup timers, restore testing (manual)
- **Standards:** 3-2-1 backup rule (3 copies, 2 media, 1 offsite)

**4.4 Service Level Expectations**
- **Policy:** Critical services ≥99.9% uptime, standard ≥99%
- **Enforcement:** HealthCheck skill, systemd service monitoring
- **Standards:** See `SERVICE_CATALOG.md` - SLE section

---

### 5. DEVELOPMENT POLICIES

**5.1 Algorithm Discipline**
- **Policy:** E2+ effort uses Algorithm phases, E1 can fast-path
- **Enforcement:** Algorithm ISC floors (E2 ≥16, E3 ≥32, etc.)
- **Standards:** See `Algorithm/v5.7.11.md`

**5.2 Honesty & No Fabrication**
- **Policy:** Identity tokens and technical claims must be retrieved, never synthesized
- **Enforcement:** IdentityValidator hook (blocks on identity-mismatch)
- **Standards:** See `PAI_SYSTEM_PROMPT.md` § Honesty

**5.3 Documentation Standards**
- **Policy:** Code/skills/agents must be documented (purpose, usage, examples)
- **Enforcement:** CreateSkill template includes SKILL.md
- **Standards:** Markdown, greppable, cross-referenced

**5.4 Testing Requirements**
- **Policy:** Changes must be verified before claiming complete
- **Enforcement:** Algorithm VERIFY phase, Reality Checker, QA Tester
- **Standards:** ISC verification = proof, not assertion

---

## Policy Hierarchy

```
Principles (7)
  ↓ inform
Policies (5 categories, ~20 policies)
  ↓ implemented via
Standards (NIST CSF, OWASP, Zero Trust - see SECURITY_STANDARDS.md)
  ↓ operationalized through
Guidelines (when to use skills, best practices - see GUIDELINES.md)
  ↓ enforced by
Tools & Hooks (automation, checks, alerts)
```

**Relationships:**
- **Principles:** Universal, never violated
- **Policies:** Specific rules derived from principles
- **Standards:** Technical specifications implementing policies
- **Guidelines:** Recommendations for applying standards
- **Tools:** Automation enforcing guidelines/standards/policies

---

## Exception Handling

**When Policy Conflict:**
1. Principle trumps policy (principles are foundational)
2. Security trumps convenience (but not usability entirely)
3. Principal decision trumps automation (human always in charge)
4. Document the exception in ISA `## Decisions` with rationale

**Exception Process:**
1. Identify which policy would be violated
2. Document why exception is necessary
3. Assess risk of exception
4. Get principal approval (if security/privacy involved)
5. Log exception with time limit (review in 30/90 days)
6. Revisit policy if exceptions frequent (policy may be wrong)

**Example:** "Normally we require ISC ≥16 for E2 work, but this task genuinely has only 12 atomic criteria. Documented in `## Decisions`, proceeding with 12."

---

## Policy Lifecycle

**Creation:**
- Policy proposed via ISA or reflection
- Rationale documented (what problem does this solve?)
- Enforcement mechanism identified (how will we know if violated?)
- Principal approval

**Review:**
- Monthly: Problem patterns (are policies being violated?)
- Quarterly: Effectiveness (are policies achieving intent?)
- Annually: Relevance (do we still need this policy?)

**Update:**
- Propose change via ISA
- Document what's changing and why
- Update enforcement mechanisms
- Communicate change (if affects workflow)
- Version bump in policy doc

**Deprecation:**
- Policy no longer needed → mark deprecated
- Enforcement turned off (but keep documented for history)
- Remove after 90 days if no issues

---

## Cross-References

**Related Frameworks:**
- `ITIL_FRAMEWORK.md` - Service management principles and practices
- `SECURITY_STANDARDS.md` - Technical security standards (NIST, OWASP, Zero Trust)
- `CORE_PROCESSES.md` - Operational processes (incident, change, access)
- `GOVERNANCE_MODEL.md` - Decision authority and escalation
- `PROBLEM_RESPONSE_PLAYBOOK.md` - Security incident response

**Related Tools:**
- `PolicyCheck.ts` - Validate compliance with policies
- `PracticeMaturity.ts` - Measure governance maturity
- `RiskRegister.ts` - Track risks and mitigations

**Related Hooks:**
- DestructiveOpGuard - Prevents destructive operations without approval
- SecurityValidator - Enforces security policies
- OutputSecretsScanner - Prevents secret leakage
- IdentityValidator - Enforces honesty policy

---

**Maintained By:** PAI System  
**Review Cadence:** Quarterly  
**Last Updated:** 2026-06-21  
**Version:** 1.0
