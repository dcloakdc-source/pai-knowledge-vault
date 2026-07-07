cd /share/PAI/Packs/pai-gitleaks
chmod +x deploy-to-nas.sh
./deploy-to-nas.sh---
artifact_type: implementation_plan
summary: |
  Integration plan for Gitleaks secret scanning tool into PAI infrastructure.
  Gitleaks will scan git repositories for exposed secrets (API keys, passwords, tokens),
  integrate with Wazuh SIEM for security monitoring, and provide automated scanning 
  capabilities through Docker containerization and CI/CD workflows.
created: 2026-01-20T09:12:37-05:00
---

# Gitleaks Integration - Implementation Plan

## 1. Executive Summary

**Objective**: Integrate [Gitleaks](https://github.com/gitleaks/gitleaks) secret detection tool into the PAI infrastructure to:
- Scan git repositories for exposed secrets (API keys, passwords, tokens, private keys)
- Integrate findings with Wazuh SIEM for centralized security monitoring
- Provide automated scanning via Docker container
- Enable pre-commit hooks for proactive secret prevention
- Support both on-demand and scheduled scanning workflows

**Scope**: This integration will create a new PAI Pack (`pai-gitleaks`) with Docker containerization, Wazuh integration, custom scanning scripts, and comprehensive documentation.

**Deployment Target**: QNAP TS-464 NAS at `\\192.168.50.52\pai\`

---

## 2. Architecture Overview

### 2.1 Components

```
pai-gitleaks/
├── Dockerfile                  # Custom Gitleaks container
├── docker-compose.yml          # Standalone deployment option
├── config/
│   ├── gitleaks.toml          # Custom detection rules
│   └── wazuh-decoder.xml      # Wazuh log decoder for Gitleaks
├── scripts/
│   ├── scan-repo.sh           # Repository scanning wrapper
│   ├── scan-all-projects.sh   # Batch scan all PAI projects
│   └── pre-commit-hook.sh     # Git pre-commit hook template
├── reports/                    # Scan output storage
├── logs/                       # Gitleaks logs for Wazuh ingestion
└── README.md                   # Pack documentation
```

### 2.2 Integration Points

1. **Docker Compose Integration**: Add Gitleaks service to `docker-compose.qnap.yml`
2. **Wazuh SIEM**: 
   - Configure log ingestion from Gitleaks JSON reports
   - Create custom decoders/rules for secret detection alerts
   - Dashboard visualization of findings
3. **PAI Projects**: Scan existing repositories in `/Projects/` directory
4. **Git Workflow**: Pre-commit hooks for proactive scanning

### 2.3 Network Architecture

```
Gitleaks Container (172.30.0.60)
    │
    ├──> Scans: /share/PAI/Projects/*
    │
    ├──> Outputs: JSON reports → /share/PAI/Packs/pai-gitleaks/reports/
    │
    └──> Logs: → Wazuh Manager (172.30.0.30) → OpenSearch Indexer (172.30.0.31)
```

---

## 3. Implementation Phases

### Phase 1: Pack Creation & Configuration ⏱️ 30 min

**Tasks**:
1. Create `\\192.168.50.52\pai\Packs\pai-gitleaks\` directory structure
2. Create custom `Dockerfile` based on `zricethezav/gitleaks:latest`
3. Create `gitleaks.toml` configuration with:
   - PAI-specific secret patterns (Gemini API keys, Wazuh credentials)
   - Extended entropy thresholds
   - Allowlist for known safe patterns
4. Create wrapper scripts for scanning workflows
5. Create `.gitleaksignore` template for false positive management

**Deliverables**:
- Complete pack directory structure
- Customized Gitleaks configuration
- Scanning automation scripts

**Verification**:
```bash
cd /share/PAI/Packs/pai-gitleaks
ls -R
cat config/gitleaks.toml
```

---

### Phase 2: Docker Container Setup ⏱️ 45 min

**Tasks**:
1. Build custom Gitleaks Docker image with:
   - Gitleaks binary
   - Git CLI tools
   - Custom scanning scripts
   - Volume mounts for PAI Projects
2. Create standalone `docker-compose.yml` for testing
3. Integrate into main `docker-compose.qnap.yml` as optional service
4. Configure:
   - Volume mounts: `/share/PAI/Projects` → `/repos` (read-only)
   - Volume mounts: `/share/PAI/Packs/pai-gitleaks/reports` → `/reports`
   - Volume mounts: `/share/PAI/Packs/pai-gitleaks/logs` → `/logs`
   - Network: `pai-network` (172.30.0.60)
   - Restart policy: `unless-stopped`

**Deliverables**:
- Working Gitleaks Docker container
- Integrated docker-compose configuration
- Persistent volume mappings

**Verification**:
```bash
docker-compose -f docker-compose.qnap.yml up -d gitleaks
docker exec pai-gitleaks gitleaks version
docker exec pai-gitleaks gitleaks detect --help
```

---

### Phase 3: Wazuh SIEM Integration ⏱️ 60 min

**Tasks**:
1. Create Wazuh custom decoder (`wazuh-gitleaks-decoder.xml`) to parse Gitleaks JSON output
2. Create Wazuh custom rules (`wazuh-gitleaks-rules.xml`) for:
   - **Critical** (90+): Exposed credentials, private keys, high-entropy secrets
   - **High** (70-89): API keys, database URLs, JWT tokens
   - **Medium** (50-69): Potential secrets, low-confidence matches
3. Configure `ossec.conf` to monitor `/share/PAI/Packs/pai-gitleaks/logs/` directory
4. Create Wazuh dashboard/visualization for Gitleaks findings
5. Set up alerting for critical secret exposures

**Deliverables**:
- Wazuh decoder for Gitleaks JSON format
- Custom alert rules with severity mapping
- Automated log ingestion configuration
- Dashboard visualization

**Verification**:
```bash
# Test decoder
docker exec pai-wazuh-manager /var/ossec/bin/wazuh-logtest < sample-gitleaks.json

# Verify rule triggering
docker exec pai-wazuh-manager grep -i gitleaks /var/ossec/logs/alerts/alerts.json
```

---

### Phase 4: Scanning Workflows ⏱️ 45 min

**Tasks**:
1. **On-Demand Scanning**:
   - Create `scan-repo.sh <repo-path>` for single repository scans
   - Create `scan-all-projects.sh` for batch scanning all PAI projects
   
2. **Scheduled Scanning**:
   - Create cron job for daily scans of all repositories
   - Configure retention policy for old reports (90 days)

3. **Pre-Commit Hook**:
   - Create `pre-commit-hook.sh` template for local git repositories
   - Provide installation instructions for developers

4. **Output Management**:
   - JSON reports stored in `/reports/` with timestamp naming
   - Failed scans logged to `/logs/scan-errors.log`
   - Summary reports sent to Wazuh

**Deliverables**:
- Three scanning workflow scripts
- Cron job configuration
- Pre-commit hook template
- Report management automation

**Verification**:
```bash
# Run manual scan
./scripts/scan-repo.sh /share/PAI/_dev_archive/Personal_AI_Infrastructure

# Check report generation
ls -lh reports/

# Verify Wazuh ingestion
docker exec pai-wazuh-manager grep "gitleaks" /var/ossec/logs/ossec.log
```

---

### Phase 5: Testing & Hardening ⏱️ 60 min

**Tasks**:
1. **Baseline Scan**: Run comprehensive scan of all PAI projects
2. **False Positive Tuning**: 
   - Review initial findings
   - Add legitimate patterns to `.gitleaksignore`
   - Adjust entropy thresholds if needed
3. **Performance Testing**:
   - Measure scan time for large repositories
   - Optimize resource limits in Docker (CPU/memory)
4. **Security Hardening**:
   - Run Gitleaks container as non-root user
   - Read-only filesystem for repositories
   - Secure report storage permissions
5. **Documentation**:
   - Create comprehensive `README.md` for the pack
   - Document common workflows (scan, remediate, allowlist)
   - Create troubleshooting guide

**Deliverables**:
- Baseline security scan results
- Optimized configuration (`.gitleaksignore`, `gitleaks.toml`)
- Performance benchmarks
- Complete documentation

**Verification**:
- All PAI projects scanned successfully
- False positive rate < 10%
- Scan time < 5 minutes for average repository
- Zero secrets exposed in production code

---

### Phase 6: Walkthrough & Handoff ⏱️ 30 min

**Tasks**:
1. Create `walkthrough.md` with:
   - Screenshots of Wazuh dashboard showing Gitleaks alerts
   - Sample scan outputs (sanitized)
   - Evidence of successful integration
2. Run demonstration scan showing end-to-end workflow:
   - Trigger scan → Generate report → Wazuh alert → Dashboard visualization
3. Update `.env` if new environment variables needed
4. Update `README-DEPLOY.md` with Gitleaks usage instructions

**Deliverables**:
- `walkthrough.md` with evidence of success
- Updated deployment documentation
- Demo recording (optional)

**Verification Checklist**:
- [ ] Gitleaks container running and healthy
- [ ] Sample repository scan completes successfully
- [ ] JSON reports generated in `/reports/`
- [ ] Wazuh receives and parses Gitleaks logs
- [ ] Critical alerts trigger correctly in Wazuh
- [ ] Dashboard displays findings
- [ ] Pre-commit hook works in test repository
- [ ] Documentation is complete and accurate

---

## 4. Risk Assessment & Mitigations

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| **False Positives** | Medium - Alert fatigue | High | Extensive tuning with `.gitleaksignore`, custom allowlist rules |
| **Performance Impact** | Low - NAS resource contention | Medium | Set Docker resource limits, run scans during off-peak hours |
| **Missed Secrets** | High - Security exposure | Low | Use strict regex patterns, enable Shannon entropy, regular rule updates |
| **Repository Access** | Medium - Permission errors | Medium | Use read-only volume mounts, test with all repo types (git, bare, archives) |
| **Wazuh Integration Failure** | Medium - No alerting | Low | Implement health checks, monitor log ingestion rates |

---

## 5. Success Criteria

✅ **Must Have**:
1. Gitleaks container deployed and running in PAI stack
2. At least one successful scan of PAI project repositories
3. Wazuh receives and displays Gitleaks findings
4. Critical secret exposures trigger high-priority Wazuh alerts
5. Documentation enables future scans without assistance

🎯 **Should Have**:
1. Automated daily scanning via cron
2. Pre-commit hook template available for developers
3. Custom PAI-specific detection rules (Gemini API, Wazuh creds)
4. Dashboard visualization in Wazuh
5. Baseline scan of all existing repositories completed

🌟 **Nice to Have**:
1. Integration with GitKraken MCP for automated PR scanning
2. Slack/email notifications for critical findings
3. Automated remediation suggestions
4. Historical trending of secret exposure over time

---

## 6. Post-Implementation Actions

1. **Week 1**:
   - Review baseline scan results
   - Remediate any discovered secrets immediately
   - Fine-tune false positive rules
   - Run daily scans for first week to establish baseline

2. **Week 2-4**:
   - Implement pre-commit hooks on active repositories
   - Create runbook for secret exposure incident response
   - Train team on Gitleaks usage and best practices
   - Monitor Wazuh alert volumes and adjust thresholds

3. **Month 2+**:
   - Expand scanning to external projects in `/Projects/External/`
   - Integrate with CI/CD pipelines (if applicable)
   - Quarterly review of detection rules and keep updated with Gitleaks releases
   - Add custom rules for new PAI services (Vaultwarden, etc.)

---

## 7. Resources & References

- **Gitleaks GitHub**: https://github.com/gitleaks/gitleaks
- **Gitleaks Documentation**: https://github.com/gitleaks/gitleaks#readme
- **Wazuh Log Data Analysis**: https://documentation.wazuh.com/current/user-manual/ruleset/index.html
- **Docker Best Practices**: https://docs.docker.com/develop/dev-best-practices/

---

## Estimated Timeline

- **Total Implementation**: ~4.5 hours
- **Testing & Hardening**: +1 hour
- **Documentation**: +0.5 hours
- **Contingency**: +1 hour
- **Grand Total**: ~7 hours

---

## Next Steps

**Awaiting Approval**: Please review this plan and approve before I proceed with Phase 1. Any specific requirements or modifications needed?

Once approved, I will:
1. Create the `pai-gitleaks` pack structure
2. Build the Docker container
3. Configure Wazuh integration
4. Run baseline scans
5. Deliver comprehensive walkthrough with evidence

