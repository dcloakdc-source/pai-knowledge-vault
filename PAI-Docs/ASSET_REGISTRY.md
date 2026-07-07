# PAI Asset Registry
**Authoritative compact reference — auto-loaded at session start**
**Last updated:** 2026-06-16 | **Full docs:** `PAI/INFRA/` | **Source of truth:** `PAI/INFRA/HOME_INVENTORY.md`
**Live device discovery:** ~55 live devices (point-in-time, root ARP-PR sweep via passwordless `pai-arpscan` wrapper) + offline-but-leased via DHCP reconciliation (61 router `dnsmasq.leases`) on 192.168.50.0/24, 2026-06-16 — see "Live Network Discovery" + "DHCP reconciliation" in `PAI/INFRA/HOME_INVENTORY.md` (re-run `bun ~/.claude/PAI/Tools/NetworkDiscovery.ts`; hourly `lan-device-watcher` alerts on new MACs).

---

## Hosts

| Host | LAN IP | Tailscale IP | Role |
|------|--------|-------------|------|
| **pai-primary** | 192.168.50.20 | 100.64.0.1 | Primary AI/service hub (VM on aorus physical) |
| **aorus** | 192.168.50.5 | 100.77.80.47 | Physical machine hosting pai-primary; GPU compute (RTX 3060 6GB + RTX 4060 8GB) |
| **QNAP TS-464** | 192.168.50.6 | — (off tailnet 2026-06-18) | Primary NAS — NFS /share/PAI → /PAI + /mnt/pai on pai-primary |
| **QNAP TS-451** | 192.168.50.8 | — | Secondary NAS |
| **m710q** | 192.168.50.10 | — | Authentik SSO + Vaultwarden |
| **Proxmox/HA** | 192.168.50.30 | — | Home Assistant OS :8123 |
| **Router** | 192.168.50.1 | 100.116.31.69 | ASUS GT-AX11000 Pro + Headscale :8080 |
| **legion** | — | 100.99.240.58 | Laptop (pai3) |
| **pai-prism** | 192.168.50.223 | — | Kitchen Prism family kiosk (Linux appliance; migrated 2026-06-14 from Windows DESKTOP-5R903RO/.43). Chrome --kiosk via Xorg autologin (acp-admin, tty1), SSH key id_ed25519_pai-primary |

> **aorus + pai-primary share the same physical NIC** — they are NOT separate compute nodes.

---

## Services on QNAP TS-464 (192.168.50.6)

| Port | Service | Auth | Scope | Notes |
|------|---------|------|-------|-------|
| 8601 | Wazuh Dashboard | Wazuh auth | LAN | SIEM UI (HTTPS); IndexerURL→wazuh-indexer:9200 |
| 1514 | Wazuh Manager (agent) | TLS | LAN | Agent log ingestion (OSSEC protocol) |
| 1515 | Wazuh Manager (authd) | TLS | LAN | Agent auto-enrollment |
| 55500 | Wazuh Manager API | Basic (wazuh/wazuh) | LAN | REST API (maps to container :55000) |
| Docker net | pai_secure-backend-net | — | Internal | 172.30.3.0/24 — Wazuh+Indexer talk here |
| Docker net | pai_dmz-net | — | LAN | 172.30.1.0/24 — Dashboard+Manager exposed here |

> **Wazuh API:** `curl -u wazuh:wazuh http://192.168.50.6:55500/security/user/authenticate`
> **Log shipping from pai-primary:** Fluent Bit → 192.168.50.6:1514 (OSSEC) or 192.168.50.6:514 (syslog)

---

## Services on pai-primary (192.168.50.20)

| Port | Service | Auth | Scope | Notes |
|------|---------|------|-------|-------|
| 3001 | Grafana | None | LAN | VictoriaMetrics + Loki datasources |
| 3010 | MetaMCP | None | Local | 8 MCP servers proxied |
| 3100 | Loki | None | Local | PAI session log store |
| 4111 | Mastra Workflow Runtime | None | Local | PAI workflow engine |
| 5432 | sovereign-db (Postgres) | DB password | **localhost only** | Primary application DB |
| 5433 | pai-memory-pg (pgvector) | DB password | **localhost only** | Vector memory DB |
| 5601 | OpenSearch Dashboards | None | LAN | SIEM UI |
| 8085 | Identus Cloud Agent | JWT | LAN | SSI/DID agent |
| 8087 | Glance Dashboard | None | LAN | System overview |
| 8090 | Open WebUI | Local auth | LAN | Ollama chat UI |
| 8091 | Plane | Plane auth | LAN | Project management |
| 8092 | Guacamole | Guacamole auth | LAN | HTML5 RDP gateway |
| 8428 | VictoriaMetrics | None | LAN | Long-term metrics |
| 8766 | PAI Command Center | None | LAN | Activity dashboard |
| 8888 | PAI Voice Server (local fallback) | PAI voice JWT / local compatibility | pai-primary | Local rollback path; Pulse now targets Legion Voicebox |
| 17493 | Legion Voicebox Sound Server | PAI voice JWT via Pulse | Legion (100.99.240.58) | Active Pulse backend; Voicebox API — POST /speak |
| 8899 | Legacy Chatterbox TTS | PAI voice JWT via Pulse | Legion (100.99.240.58) | Superseded by Voicebox; current service observed failed/closed |
| 8893 | Legacy Whisper STT | None (LAN) | Legion (100.99.240.58) | Superseded/closed during Voicebox deployment |
| 8890 | PAI Functions API | None | LAN | JIT credential gateway |
| 8891 | PAI Memory MCP SSE | None | Local | Memory MCP bridge |
| 8892 | PAI Ollama A2A Bridge | None | LAN | Ollama A2A protocol |
| 9000 | Authentik SSO (proxy) | Authentik | LAN | SSO proxy; canonical host: m710q |
| 9001 | Step-CA | mTLS | Local | Internal PKI |
| 9200 | OpenSearch | None (security off) | **localhost only** | SIEM data store |
| 11434 | Ollama | None | LAN | Local LLM inference |
| 8123 | Home Assistant | HA auth | LAN | Smart home (on Proxmox VM at .30) |

---

## Key Paths & Environment

| Variable / Path | Value | Notes |
|-----------------|-------|-------|
| `PAI_DIR` | `/home/duane/.claude` | Primary PAI working directory |
| `PROJECTS_DIR` | `/PAI/` | NFS-mounted project root (symlink from /share/PAI) |
| NFS share | `/share/PAI` → `/PAI` | QNAP TS-464 fast path |
| Backup mount | `/mnt/pai` | Secondary NAS mount |
| Secrets | `~/.config/PAI/secrets.env` | API keys, tokens |
| PKI signing key | `~/.config/PAI/pki/signing.jwk.json` | JWT signing |
| PAI config | `/share/PAI/Source/Projects/pai-opencode/` | OpenCode PAI project |
| OpenCode symlink | `~/.opencode` → `/share/PAI/Source/Projects/pai-opencode/.opencode` | OpenCode config |
| OpenSearch data | localhost:9200 | Never expose to LAN |
| Postgres | localhost:5432 (sovereign), localhost:5433 (pgvector) | Never expose to LAN |

---

## PAI Systemd Services (pai-primary)

| Service | Port | Status | Role |
|---------|------|--------|------|
| pai-pulse | 31337 | active | Canonical engine-facing voice compatibility route: `/notify`; forwards to Legion Voicebox at `100.99.240.58:17493` |
| pai-voiceserver | 8888 | fallback | Canonical current unit: `pai-voiceserver.service`; local rollback path while Legion voice soaks |
| pai-functions | 8890 | active | Canonical current unit: `pai-functions.service`; JIT credential gateway; `/v1/voice` forwards through Pulse |
| pai-command-center | 8766 | active | Web activity dashboard |
| pai-memory-sse | 8891 | active | Memory MCP SSE bridge |
| pai-ollama-bridge | 8892 | active | Ollama A2A protocol |
| pai-switchboard-bridge | — | active | Inter-agent messaging |
| pai-inbox-watchdog | — | active | Event-driven triggers |
| pai-mastra | 4111 | active | Workflow runtime |
| pai-litestream | — | active | DB replication |

---

## Agent Roster

| Agent | ID | Interface | Model | Notes |
|-------|-----|-----------|-------|-------|
| Nova (primary) | PNC | Claude Code CLI | Sonnet 4.6 | This session's DA |
| Antigravity CLI | AGT-002 | `agy` bash (PAI `agy` shim) | Antigravity CLI v1.0.2 | Research agent; run with `agy --dangerously-skip-permissions -p "..."` |
| PAI-OpenCode | AGT-003 | OpenCode CLI (`opai`) | opencode/big-pickle | Complex code tasks |
| Switchboard-Broker | AGT-004 | Service | — | Real-time message routing between agents |
| Antigravity IDE | AGT-005 | Antigravity IDE (VS Code fork) | Gemini 2.x (in-IDE) | Visual Gemini workspace, research board, planning artifacts |
| Ollama models | — | 192.168.50.20:11434 | llama3.2:3b / gemma2:9b / qwen2.5:7b | Free local inference |

---

*To update: edit `PAI/INFRA/HOME_INVENTORY.md` (authoritative), then sync changes here.*
*Full infrastructure docs: `PAI/INFRA/TOPOLOGY.md`, `PAI/INFRA/SERVICE_MATRIX.md`, `PAI/PAI_INFRASTRUCTURE_DESIGN.md`*

<!-- AUTOGEN:catalogue START -->

## Agents (auto-synced)

_33 subagent personas — `agents/*.md`_

| Agent | Description |
|-------|-------------|
| AgentsOrchestrator | Autonomous pipeline manager for multi-agent software workflows. Converts specs into task DAGs, routes to specialist age… |
| Algorithm | Expert in creating and evolving Ideal State Criteria (ISC) as part of the PAI Algorithm's core principles. Specializes… |
| AntigravityIDE | Antigravity IDE (Google VS Code fork, v1.107.0) — visual Gemini workspace. NOT the `agy` CLI. Use for research boards,… |
| Architect | Elite system design specialist with PhD-level distributed systems knowledge and Fortune 10 architecture experience. Cre… |
| Artist | Visual content creator. Called BY Media skill workflows only. Expert at prompt engineering, model selection (Flux 1.1 P… |
| BackendDev | Backend implementation specialist. Writes server-side code, APIs, database logic, and scripts. Works from clear require… |
| BrowserAgent | Parallel headless browser automation agent using Playwright CLI. Navigates pages, interacts with elements, extracts dat… |
| ClaudeResearcher | Academic researcher using Claude's WebSearch. Called BY Research skill workflows only. Excels at multi-query decomposit… |
| CodeReviewer | Read-only code quality reviewer focused on security, performance, and maintainability. Reviews diffs and changed files.… |
| CodeReviewer-Agency | Thorough, constructive code reviewer focused on correctness, security, maintainability, and performance. Uses priority… |
| CodexResearcher | Remy - Eccentric, curiosity-driven technical archaeologist who treats research like treasure hunting. Consults multiple… |
| DeepResearcher | Deep researcher for comprehensive investigations. Called BY Research skill workflows only. Excels at multi-query decomp… |
| Designer | Elite UX/UI design specialist with design school pedigree and exacting standards. Creates user-centered, accessible, sc… |
| Engineer | Elite principal engineer with Fortune 10 and premier Bay Area company experience. Uses TDD, strategic planning, and con… |
| GeminiResearcher | Multi-perspective researcher using Google Gemini. Called BY Research skill workflows only. Breaks complex queries into… |
| GrokResearcher | Johannes - Contrarian, fact-based researcher using xAI Grok API. Specializes in unbiased analysis of social/political i… |
| IncidentResponseCommander | Production incident management specialist. Runs structured SEV1-SEV4 response with explicit role assignment, blameless… |
| Intern | Use this agent when you need an exceptionally intelligent, high-agency generalist to solve complex problems. 176 IQ gen… |
| LocalAnalyst | Persona-flavored Claude sub-agent ("Adam Qwen") for structured analysis, JSON extraction, CVSS scoring, and tabular wor… |
| LocalReasoner | Persona-flavored Claude sub-agent ("Callum Seek") for multi-step reasoning, deep analysis, and tradeoff evaluation. NOT… |
| LocalRunner | Persona-flavored Claude sub-agent ("Eric Llama") for fast classification and sentiment analysis. NOTE — runs on Claude… |
| LocalSummarizer | Persona-flavored Claude sub-agent ("Antoni Gemma") for document analysis, large-context synthesis, and summarization. N… |
| MCPBuilder | MCP server design and implementation specialist. Obsessed with tool naming clarity — agents select tools by name alone,… |
| Pentester | Offensive security specialist. Called BY Webassessment skill workflows only. Performs vulnerability assessments, penetr… |
| PerplexityResearcher | Ava - Investigative analyst using Perplexity API for web research. Called BY Research skill workflows only. Triple-chec… |
| ProductManager | Read-only requirements analyst. Explores codebase and clarifies user stories, acceptance criteria, and scope. Cannot wr… |
| QATester | Quality Assurance validation agent that verifies functionality is actually working before declaring work complete. Uses… |
| RealityChecker | Skeptical quality gate agent. Default verdict is NEEDS WORK — requires overwhelming evidence for production readiness.… |
| SoftwareArchitect | System design specialist focused on maintainable, domain-aligned architectures. Uses bounded contexts, trade-off matric… |
| TeamLead | Engineering team orchestrator. Breaks large features into parallel workstreams and delegates to specialist sub-agents.… |
| Tester | QA specialist focused on test coverage, edge cases, and regression prevention. Writes and runs tests. Invoke with @Test… |
| UIReviewer | User story validation agent using Playwright CLI. Accepts a structured story (URL + steps + assertions), executes each… |
| Writer | Emma Hartley - Technical storyteller who translates complexity into narrative. Content creation, docs, blog posts, tech… |

## RBAC Identities (auto-synced)

_9 identities — `PAI/agents.yml`; tiers sourced from RBAC_SCHEMA.md @ 2026-06-18T01:33:31.341Z_

| ID | Name | Role | Tier | Secrets |
|-----|------|------|------|---------|
| AGT-001 | PAI Nova (Claude) | pai-admin | 0 | * |
| AGT-002 | PAI Nova (Gemini) | pai-agent-privileged | 1 | 3 |
| AGT-003 | PAI Nova OpenCode (opai) | pai-agent-privileged | 1 | 3 |
| AGT-004 | PAI MCP Server | pai-agent-standard | 2 | 0 |
| AGT-005 | PAI Antigravity (Gemini CLI) | pai-agent-privileged | 1 | 3 |
| AGT-006 | PAI Command Center | pai-admin | ? | * |
| AGT-007 | PAI Nova Ollama (PNO) | pai-agent-standard | 2 | 0 |
| AGT-008 | PAI Nova Codex (PNX) | pai-agent-privileged | 1 | 3 |
| USR-001 | Duane | pai-admin | 0 | * |

_Secrets column shows COUNT or `*` — never secret values._

## Tools (auto-synced)

_417 entry-point scripts in `PAI/Tools/` (top-level; libs/adapters in subdirs excluded)_

**TypeScript** (345):

`ActiveContext.ts` · `ActivityFeed.ts` · `ActivityParser.ts` · `ActivityTimeline.ts` · `AddBg.ts` · `Agenda.ts` · `AgentLedger.ts` · `AgentLogger.ts` · `AgentLoopMonitor.ts` · `AgentMatrixTui.ts` · `AgentMessenger.ts` · `AgentMessengerDaemon.ts` · `AgentPoW.ts` · `AgentWatchdog.ts` · `AlgorithmPhaseReport.ts` · `AlgorithmReplay.ts` · `AlgorithmSkipAudit.ts` · `AlgorithmVersionTriggers.ts` · `AlignmentTracker.ts` · `AnalyzePermissions.ts` · `AntigravityWorker.ts` · `ApprovalGate.ts` · `ArchiveWork.ts` · `AssetInventory.ts` · `Audit.ts` · `AuditAgentLogging.ts` · `AutonomyRunner.ts` · `BackgroundAgentManager.ts` · `Banner.ts` · `BannerMatrix.ts` · `BannerNeofetch.ts` · `BannerPrototypes.ts` · `BannerRetro.ts` · `BannerTokyo.ts` · `Batch2Distiller.ts` · `BenchmarkLLM.ts` · `BrainIngest.ts` · `BudgetReport.ts` · `BuildCLAUDE.ts` · `BulkIndexAutoMemory.ts` · `BulkIndexICM.ts` · `BulkIndexKnowledge.ts` · `BulkIndexMemoirs.ts` · `BulkIndexPAIMemoryDB.ts` · `BulkIndexPAIUser.ts` · `BulkIndexPRDs.ts` · `BulkIndexReflections.ts` · `BulkMediaExtract.ts` · `Checkpoint.ts` · `ClaudeJsonGuard.ts` · `Conductor.ts` · `ContextPrimer.ts` · `CorrelationEngine.ts` · `Cost.ts` · `Council5x5.ts` · `CouncilSession.ts` · `CreatePostmortem.ts` · `CreateReflection.ts` · `CrossDomainLinker.demo.ts` · `CrossDomainLinker.test.ts` · `CrossDomainLinker.ts` · `CrossEngineBudget.ts` · `CrossVendorAudit.ts` · `DaemonAggregator.ts` · `DataDistiller.ts` · `DeadHooks.ts` · `DelegateSINA.ts` · `DeviceRegistry.ts` · `DocCheck.ts` · `DocVault.ts` · `DoclingIngest.ts` · `EngineBench.ts` · `EngineDashboard.ts` · `EngineMeter.ts` · `EngineRouter.test.ts` · `EngineRouter.ts` · `EntityGraph.ts` · `EventsServer.ts` · `ExtractTranscript.ts` · `FailureCapture.ts` · `FamilyDigest.ts` · `FeatureRegistry.ts` · `FeedMonitor.ts` · `FeedbackProcessor.ts` · `FleetReaper.ts` · `FocusManager.test.ts` · `FocusManager.ts` · `FocusPomodoro.ts` · `FunctionsAPIClient.ts` · `GTD.ts` · `GeminiEmbed.ts` · `GenerateCrossEngineSkillIndex.ts` · `GenerateHandoff.ts` · `GenerateServiceCatalog.ts` · `GenerateSkillIndex.ts` · `GenerateSkillSchemas.ts` · `GetCounts.ts` · `GetTranscript.ts` · `GitHubDiscovery.ts` · `HAAction.ts` · `HAQuery.ts` · `HerdrAgentWatcher.ts` · `HerdrCockpit.ts` · `HerdrSocket.ts` · `HerdrWebLauncher.ts` · `HistoricalDistiller.ts` · `HonestyReport.ts` · `HonestyValidator.ts` · `HookDispatchSmoke.ts` · `HybridInference.ts` · `ISCVerifier.test.ts` · `ISCVerifier.ts` · `ITILMetricsCollector.ts` · `ITILMonthlyReview.ts` · `InboxAutoProcessor.ts` · `InboxWatchdog.ts` · `Inference.ts` · `InfraJIT.ts` · `InsightsReport.ts` · `IntegrityMaintenance.ts` · `InteractivePrompt.test.ts` · `InteractivePrompt.ts` · `InternalMail.ts` · `KioskRebuild.ts` · `KioskStatus.ts` · `KnowledgeHarvester.ts` · `LearningPatternSynthesis.ts` · `LlamaCppLoadBalancer.ts` · `LlamaCppPilot.ts` · `LoadSkillConfig.ts` · `LogHealth.ts` · `LogProblem.ts` · `MatrixUpdater.ts` · `McpProfiles.ts` · `MediaWorkflow.ts` · `Memory.ts` · `MemoryACL.ts` · `MemoryAppend.ts` · `MemoryBackfill.ts` · `MemoryConflictResolver.ts` · `MemoryEmbed.ts` · `MemoryIngest.ts` · `MemoryPromoter.ts` · `MemoryQuery.ts` · `MemoryRecallStats.ts` · `MemoryStatus.ts` · `MetricsExporter.ts` · `ModelBenchmark.ts` · `ModelProfile.ts` · `ModelRouter.ts` · `MonarchClient.ts` · `MorningBriefing.ts` · `MorningRitual.ts` · `NeofetchBanner.ts` · `NetworkCapture.ts` · `NetworkDiscovery.ts` · `NotifyPNC.ts` · `OTelExporter.ts` · `ObservabilityServer.ts` · `ObservabilityTransport.ts` · `OllamaMode.ts` · `OmniPulseHonestyAudit.ts` · `OpaiDispatch.ts` · `OpaiQueue.ts` · `OpenSpecBridge.ts` · `OpinionTracker.ts` · `OrchestrationTest.ts` · `OwuiSync.ts` · `PAIDirCatalog.ts` · `PAIDirMigrate.ts` · `PAILogo.ts` · `PaiQuarantine.ts` · `ParallelEngineExecution.ts` · `ParameterTuner.ts` · `PathResolver.demo.ts` · `PathResolver.migration-example.ts` · `PathResolver.test.ts` · `PathResolver.ts` · `PatternValidator.ts` · `PeerReview.ts` · `PentestNormalizer.ts` · `PipelineMonitor.ts` · `PipelineOrchestrator.ts` · `PlanChecker.ts` · `PlanningOracle.ts` · `PolicyCheck.ts` · `PracticeMaturity.ts` · `PreConnectionPrompt.ts` · `PreviewMarkdown.ts` · `ProblemMetrics.ts` · `ProjectManager.ts` · `QualitySweep.ts` · `RBACSecurityAudit.ts` · `RLSDOracleStatus.ts` · `RagasEval.ts` · `RampClaimCheck.ts` · `RampPrecision.ts` · `RebuildPAI.ts` · `ReceiveFromMe.ts` · `RecordRouter.ts` · `ReflectionHealth.ts` · `ReflectionInjector.ts` · `ReflectionMiningCheck.ts` · `ReflectionReview.ts` · `ReflectionSynthesizer.ts` · `RegistryAudit.ts` · `RelationshipReflect.ts` · `RemoveBg.ts` · `RiskRegister.ts` · `RotateFlows.ts` · `RouterClient.ts` · `RunbookGen.ts` · `RuntimeAdapter.demo.ts` · `RuntimeAdapter.test.ts` · `RuntimeAdapter.ts` · `SSEServer.ts` · `Scheduler.ts` · `SearxngDiscover.ts` · `SecretScan.ts` · `SecretScanQuick.ts` · `Security.ts` · `SecurityFilter.ts` · `SecurityReviewCheck.ts` · `SecurityRouter.ts` · `Seeds.ts` · `SemanticRecall.ts` · `SendToMe.ts` · `ServiceDiscovery.example.ts` · `ServiceDiscovery.test.ts` · `ServiceDiscovery.ts` · `SessionBus.ts` · `SessionHarvester.ts` · `SessionIndex.ts` · `SessionJournal.ts` · `SessionLog.ts` · `SessionProgress.ts` · `SessionRegistry.ts` · `SessionReport.ts` · `SessionTracker.ts` · `SignalWatcher.ts` · `SkillAdapterBacklog.ts` · `SkillAdapterContract.ts` · `SkillAdapterConvert.ts` · `SkillAdapterPreservationAudit.ts` · `SkillCatalogGovernance.ts` · `SkillConsolidationArchive.ts` · `SkillConsolidationManifest.ts` · `SkillConsolidationProbe.ts` · `SkillGapDetector.ts` · `SkillGovernanceValidate.ts` · `SkillPortfolioAudit.ts` · `SnykToAssetRegistry.ts` · `SolveTestPoW.ts` · `SplitAndTranscribe.ts` · `StackAutoRecover.ts` · `StackBackup.ts` · `StackBackupCron.ts` · `StackBackupVerify.ts` · `StackMonitor.ts` · `StackRestart.ts` · `StackRestoreTest.ts` · `StackStatus.ts` · `SubagentSpendReport.ts` · `SudoSession.ts` · `SwitchProvider.ts` · `SwitchboardClient.ts` · `SyncFamilyAgendaFromLive.ts` · `SystemAudit.ts` · `TaskQueue.ts` · `TestFeedback.ts` · `TestHandshake.ts` · `TestOllamaConfig.ts` · `ThemeDecayCheck.ts` · `TranscriptParser.ts` · `TuningPipeline.ts` · `TxtaiIndex.ts` · `UpgradeCheck.ts` · `UsageProbe.ts` · `UserModeler.ts` · `ValidateHybridQuality.ts` · `ValidateSkillStructure.ts` · `VaultMirror.ts` · `VaultProxy.ts` · `VerifyNotesSignature.ts` · `VoiceRouter.ts` · `WikilinkGraph.ts` · `WisdomCrossFrameSynthesizer.ts` · `WisdomDomainClassifier.ts` · `WisdomFrameUpdater.ts` · `WorkLogger.ts` · `Workstreams.ts` · `YouTubeApi.ts` · `algorithm.ts` · `budget-shared.test.ts` · `budget-shared.ts` · `bus-prepend.ts` · `cf-access.ts` · `container-healthprobe.ts` · `crawl4ai.ts` · `deadman-degraded.ts` · `deadman.ts` · `distill.ts` · `egress-allowlist.ts` · `extract-pai-dir-settings.ts` · `fetch-secret.ts` · `google-apps-ingest.ts` · `harvest.ts` · `herdr-engine-launch.ts` · `iam-audit.ts` · `inference-matrix-validate.ts` · `jit-cred.ts` · `merge-m710q-databases.ts` · `okf-discover.ts` · `okf-export.ts` · `okf-import.ts` · `okf-mcp.ts` · `okf-promote.ts` · `ollama-archetype-agents.ts` · `opencode-exporter.ts` · `pai-cockpit-server.ts` · `pai-egress-provenance.ts` · `pai-egress-proxy.ts` · `pai-egress-relay.ts` · `pai-fs.ts` · `pai-guard.ts` · `pai-infra.ts` · `pai-mcp-server.ts` · `pai-perm.ts` · `pai-provenance.ts` · `pai-tailscale.ts` · `pai.ts` · `process-m710q-memory.ts` · `purge-harvest-noise.ts` · `qts-api.ts` · `reconcile.ts` · `redact-log-secrets.ts` · `reddit.ts` · `research-catalog.ts` · `send-to-me.ts` · `skill-exec.ts` · `substrate-ingest-cheatsheet.ts` · `substrate-ingest.ts` · `theme-extractor.ts` · `whereisduane.ts` · `youtube-ingest.ts`

**Shell** (48):

`FixRemainingSudo.sh` · `InfraBackup.sh` · `InteractivePrompt.integration.test.sh` · `NasBackup.sh` · `OpaiWatchdog.sh` · `OpaiWorker.sh` · `PackageUpdates.sh` · `PiGsdWrapper.sh` · `PostRebootSetup.sh` · `QdrantBackup.sh` · `ReindexWeekly.sh` · `SnykRecurringScan.sh` · `SwitchboardClient.test.sh` · `authentik-rbac-setup.sh` · `backup-watchdog.sh` · `bw-session.sh` · `check-xrdp-audio.sh` · `collect-session-data.sh` · `council-on-training-done.sh` · `docker-volume-backup.sh` · `engine-wrappers.sh` · `fetch-secret.sh` · `gpu_status.sh` · `herdr-notify.sh` · `lora-tune.sh` · `memory-nas-backup.sh` · `memory-shim.sh` · `notebooklm-pai.sh` · `ollama-disk-cleanup.sh` · `pai-antigravity.sh` · `pai-dashboard.sh` · `pai-engine-statusline.sh` · `pai-gemini-statusline.sh` · `pai-mcp-enable.sh` · `pai-mcp-sync.sh` · `pai-secret.sh` · `pai-statusline-core.sh` · `pai-tmux-start.sh` · `pai-tmux-tokens.sh` · `prism-nas-backup.sh` · `qts-api.sh` · `restic-backup.sh` · `sqlite-snapshot-backup.sh` · `system_health.sh` · `vllm-benchmark.sh` · `vllm-install.sh` · `voice-capture.sh` · `voice-notify-ssh.sh`

**Python** (24):

`SwitchboardBridge.py` · `SwitchboardBridge.test.py` · `TelemetryAnalyzer.py` · `agent-job-routing.py` · `extract-transcript.py` · `flatten-refs.py` · `memory_embed.py` · `merge_rlsd_adapters.py` · `nightly_brief.py` · `org-rbac-live-enforcer.py` · `org-session-manager.py` · `owui_pai_tools.py` · `pai-mcp-server.py` · `rlsd_benchmark.py` · `rlsd_benchmark_local.py` · `rlsd_etl.py` · `rlsd_etl_variance.py` · `skill-model-tuning-review.py` · `switchboard-rbac-router.py` · `telegram-bot.py` · `train_rlsd.py` · `train_tpo.py` · `validate-archetype-engagement.py` · `validate-org-rbac.py`

## Skills (auto-synced)

_122 skills — `skills/*/SKILL.md`_

| Skill | Description |
|-------|-------------|
| Agents | Dynamic agent composition and management system. USE WHEN user says create custom agents, spin up custom agen… |
| agents-sdk | Build AI agents on Cloudflare Workers using the Agents SDK. USE WHEN creating stateful agents, durable workfl… |
| AnnualReports | Annual security report aggregation and analysis. USE WHEN annual reports, security reports, threat reports, i… |
| AntigravitySkill | Canonical dispatch for Antigravity CLI (`agy`) — web research, factual lookup, and parallel search via the `a… |
| ApertureOscillation | 3-pass scope oscillation holding the question constant while shifting scope — narrow/tactical, wide/strategic… |
| Aphorisms | Aphorism management. USE WHEN aphorism, quote, saying. SkillSearch('aphorisms') for docs. |
| Apify | Social media scraping, business data, e-commerce via Apify actors. USE WHEN Twitter scraping, Instagram scrap… |
| Art | Complete visual content system. USE WHEN art, illustrations, diagrams, visualizations, mermaid, flowchart, he… |
| ArXiv | Search and retrieve arXiv papers by topic, category, or ID, with AlphaXiv AI-generated overviews. arXiv Atom… |
| AudioEditor | Audio/video editing pipeline: Whisper word-level transcription → Claude segment classification (KEEP / CUT_FI… |
| bbc | Tune into the "BBC channel" — the persistent context anchor for Duane's BBC erotic-agency essay + FetLife thr… |
| BeCreative | Extended thinking mode. USE WHEN be creative, deep thinking, deep thinking, extended reasoning. SkillSearch('… |
| BitterPillEngineering | Audits any AI instruction set for over-prompting via the core test: would a smarter model make this rule unne… |
| BrightData | Progressive URL scraping. USE WHEN Bright Data, scrape URL, web scraping tiers. SkillSearch('brightdata') for… |
| Browser | Debug-first browser automation with always-on visibility. Console logs, network requests, and errors captured… |
| BulkMediaExtract | Browser-tab media extractor via Chrome DevTools Protocol. USE WHEN download media from open tabs, extract ima… |
| bun-file-io | USE WHEN file operations, reading files, writing files, scanning directories, deleting files, Bun file APIs,… |
| ClaudeCodeFeatures | Reference skill for Claude Code platform features across recent versions (v2.1.72–v2.1.92+). USE WHEN user as… |
| cloudflare | Deploy Cloudflare Workers/Pages. USE WHEN Cloudflare, worker, deploy, Pages, MCP server. Covers Workers, Page… |
| cloudflare-email-service | Send and receive transactional emails with Cloudflare Email Service (Email Sending + Email Routing). Use when… |
| Context Search | Search prior work to add context to any request — search PRDs, git history, session names, and work directori… |
| ContextManagement |  |
| Council | Claude-persona multi-agent debate. USE WHEN quick consensus check, fast 1-round perspectives, role-based disc… |
| CreateCLI | Generate TypeScript CLIs. USE WHEN create CLI, build CLI, command-line tool. SkillSearch('createcli') for doc… |
| CreateSkill | Create and validate skills. USE WHEN create skill, new skill, skill structure, canonicalize. SkillSearch('cre… |
| Daemon | Manage the public daemon profile — a living digital representation of what you're working on, thinking about,… |
| Debate | Fast two-model debate — Claude vs Gemini, Ollama synthesizes. USE WHEN you want exactly two sides argued quic… |
| Delegation | Parallelize work via six patterns: built-in agents (Engineer/Architect/Algorithm/Explore/Plan via Task), work… |
| DeployOrchestrator | > |
| DesignCritique | Systematic design review against Nielsen's 10 heuristics and universal design laws. Evaluates UI components,… |
| DesignThinking | Structured design thinking process for problem framing, ideation, and solution validation. Guides through the… |
| Documents | Document processing. USE WHEN document, process file. SkillSearch('documents') for docs. |
| durable-objects | Create and review Cloudflare Durable Objects. Use when building stateful coordination (chat rooms, multiplaye… |
| Evals | Agent evaluation framework based on Anthropic's best practices. USE WHEN eval, evaluate, test agent, benchmar… |
| Excalidraw | > |
| ExecutiveSummary | Generate executive summaries from security assessments with automated translation, risk scoring, and quality… |
| ExploreThemes | Surface PAI improvement themes and catalog-worthy research topics using 8 structural/historical/semantic meth… |
| ExtractWisdom | Dynamic wisdom extraction that adapts sections to content. USE WHEN extract wisdom, analyze video, analyze po… |
| Fabric | Intelligent prompt pattern system with 240+ specialized patterns for content analysis, extraction, and transf… |
| FirstPrinciples | First principles analysis. USE WHEN first principles, fundamental assumptions, break down to fundamentals, as… |
| GitMemory | Implement OpenCode/Align AI's git-based agent memory pattern — store agent context in commit.md and branch.md… |
| graphify | Knowledge graph generator for files and folders. USE WHEN /graphify, graphify, build knowledge graph, map cod… |
| GSDDiscussPhase | > |
| GSDExecutePhase | > |
| GSDNewProject | > |
| GSDPlanPhase | > |
| GSDVerifyWork | > |
| Ideate | Evolutionary ideation engine — loop-controlled multi-cycle generation through 9 phases (CONSUME, DREAM 0.9, D… |
| InfoSecRiskAssessment | Council-ratified Cyber/InfoSec Risk Assessment methodology (Evidence-Graded STRIDE) for vendors, SaaS, and AI… |
| Interceptor | Real Chrome automation via the Interceptor extension — controls the actual browser from inside (zero CDP fing… |
| Interview | Phased conversational interview across all PAI context files via InterviewScan.ts (orders targets by PHASE, a… |
| ISA | Owns the Ideal State Artifact (ISA) — the single document that articulates, drives, and verifies a thing's id… |
| IterativeDepth | Multi-angle iterative exploration for deeper ISC extraction. USE WHEN iterative depth, deep exploration, mult… |
| KeyRotation | Incident-response key rotation with a redeploy map — knows every secret store, every consumer, and what must… |
| Knowledge | Manages the PAI Knowledge Archive — a typed graph across People, Companies, Ideas, Research. Operations: sear… |
| Loop | Iterative improvement loop — revisit and refine a target across multiple Algorithm cycles toward an ideal sta… |
| McpProfiles | Per-session MCP profile manager with Herdr auto-detect, tmux sync, and Pulse/Conductor integration. USE WHEN… |
| MediaWorkflow | Unified media download, review, save, and analysis workflow. Replaces fragmented browser extensions (Video Do… |
| MercuryFIM | > |
| Migrate | Intakes external content (.md/.txt, stdin, TELOS/MEMORY/KNOWLEDGE dirs, CLAUDE.md/.cursorrules/Custom Instruc… |
| ModelTraining | Autonomous local model training with smart promotion gates. Manages full pipeline from service control throug… |
| MultiEngineSynthesis | Formal cross-engine consolidation methodology for any multi-agent assessment, review, or research output. USE… |
| NotebookLM | Google NotebookLM integration via notebooklm-py. USE WHEN user wants to analyze content with NotebookLM, crea… |
| OllamaSkill | Local-first inference dispatch via PAI/Tools/Inference.ts — routes to Ollama for cheap task types (classifica… |
| openspec-apply-change | Implement tasks from an OpenSpec change. Use when the user wants to start implementing, continue implementati… |
| openspec-archive-change | Archive a completed change in the experimental workflow. Use when the user wants to finalize and archive a ch… |
| openspec-explore | Enter explore mode - a thinking partner for exploring ideas, investigating problems, and clarifying requireme… |
| openspec-propose | Propose a new change with all artifacts generated in one step. Use when the user wants to quickly describe wh… |
| Optimize | Autonomous optimization loop — hill-climb any target. Code with metrics, or skills/prompts/agents with LLM-as… |
| OSINT | Open source intelligence gathering. USE WHEN OSINT, due diligence, background check, research person, company… |
| PAI |  |
| PAIBridge | > |
| PAISecurityAudit | > |
| PAIUpgrade | Extract system improvements from content AND monitor external sources (Anthropic ecosystem, YouTube). USE WHE… |
| ParentalControl | > |
| Parser | Parse general URLs, files, articles, transcripts, and PDFs into structured JSON. USE WHEN parse, extract to J… |
| PhaseRoadmap | > |
| PIGSD | > |
| PlaywrightCli | Build and use Indy Dev Dan's 4-layer Claude Code Playwright CLI skill architecture: Skills → Agents → Justfil… |
| pno-local-triage | PNO/Ollama local triage wrapper for cheap classification, structured JSON extraction, summarization, ticket t… |
| PrivateInvestigator | Ethical people-finding. USE WHEN find person, locate, reconnect, people search, skip trace. SkillSearch('priv… |
| ProgrammaticTools | Apply Anthropic's Programmatic Tool Calling 2.0 pattern — agents write reusable code chains instead of indivi… |
| ProjectState | > |
| Prompting | Meta-prompting system for dynamic prompt generation using templates, standards, and patterns. USE WHEN meta-p… |
| PromptInjection | Prompt injection testing. USE WHEN prompt injection, jailbreak, LLM security, AI security assessment, pentest… |
| PRWorkflow | > |
| ReceiveFromMe | Monitor and process content submitted from Duane's laptop/mobile via the NAS Shared-Inbox. USE WHEN /receive,… |
| Recon | Security reconnaissance. USE WHEN recon, reconnaissance, bug bounty, attack surface. SkillSearch('recon') for… |
| RedTeam | Adversarial analysis with 32 agents. USE WHEN red team, attack idea, counterarguments, critique, stress test.… |
| Remotion | Programmatic video creation with React. USE WHEN video, animation, motion graphics, video rendering, React vi… |
| Research | Comprehensive research, analysis, and content research system. USE WHEN user says 'research' (ANY form - this… |
| ResearchPipeline | Combined YouTube + NotebookLM research pipeline. USE WHEN user wants to research a topic end-to-end — search… |
| RootCauseAnalysis | Structured incident investigation (Toyota, Ishikawa, Reason's Swiss Cheese, Gano's Apollo, Google SRE blamele… |
| Sales | Sales workflows. USE WHEN sales, proposal, pricing. SkillSearch('sales') for docs. |
| sandbox-sdk | Build sandboxed applications for secure code execution. USE WHEN building AI code execution, code interpreter… |
| Science | Universal thinking and iteration engine based on the scientific method. USE WHEN user says "think about", "fi… |
| SECUpdates | Security news aggregation. USE WHEN security news, security updates, breaches, what's new in security, securi… |
| SecurityTriage | > |
| SendToMe | Send a file from pai-primary to Duane's laptop via the NAS Shared-Inbox. USE WHEN Duane says "send to me", "s… |
| SettingsLab | Self-improving Claude Code settings system. Plan a session by activity (plan), recommend settings for a behav… |
| Simplify | Review recently changed code for opportunities to improve reuse, quality, and efficiency — then fix any issue… |
| Spatial3D | Turn a structured floor-plan spec into a real interactive 3D model (glTF) using Blender headless, viewable/em… |
| SystemsThinking | Structural analysis of complex systems (Meadows, Senge, Forrester, Ackoff, Santa Fe tradition). Five workflow… |
| Telos | Life OS and project analysis. USE WHEN TELOS, life goals, projects, dependencies, books, movies. SkillSearch(… |
| Think | Unified debate/council router. Analyzes the question and routes to the right mechanism automatically. USE WHE… |
| ThreadReply | Draft charitable-but-firm replies to comments in an online thread (FetLife, forums, social) that encourage op… |
| TRVRM | Threat, Risk, Vulnerability and Remediation Management - Complete operational framework for vulnerability lif… |
| ui-ux-pro-max | UI/UX design intelligence. USE WHEN ui, ux, design, ui/ux, frontend, website, app, dashboard, component, UI i… |
| UnifiedCouncil | Full 9-model council debate with session persistence, claim tracking, staged dispatch, and decision voting. U… |
| USMetrics | US economic indicators. USE WHEN GDP, inflation, unemployment, economic metrics, gas prices. SkillSearch('usm… |
| UXAudit | Comprehensive pre-delivery UX audit against all UI/UX standards. Walks every item in the quality checklist ac… |
| web-perf | Analyzes web performance using Chrome DevTools MCP. Measures Core Web Vitals (LCP, INP, CLS) and supplementar… |
| WebAssessment | Web security assessment. USE WHEN web assessment, pentest, security testing, vulnerability scan. SkillSearch(… |
| Webdesign | Design and integrate web interfaces using Anthropic's Claude Design (claude.ai/design) as the primary engine,… |
| WebMcp | Implement Google's WebMCP pattern — web apps expose MCP tools declaratively via HTML attributes, making any w… |
| WorkCommand | Instant WORK session recall command. USE WHEN /work command, /w command, list recent work sessions, search WO… |
| workers-best-practices | Reviews and authors Cloudflare Workers code against production best practices. USE WHEN writing new Workers,… |
| WorldThreatModel | Persistent world-model harness stress-testing ideas, strategies, and investments across 11 time horizons (6 m… |
| wrangler | Cloudflare Workers CLI for deploying, developing, and managing Workers, KV, R2, D1, Vectorize, Hyperdrive, Wo… |
| WriteStory | Layered fiction writing system using Will Storr's storytelling science and rhetorical figures. USE WHEN write… |
| YouTube | YouTube research skill using yt-dlp. USE WHEN user wants to search YouTube, analyze YouTube videos, find tren… |
| YouTubeSubscriptions | YouTube subscription management and monitoring. USE WHEN user says "youtube subscriptions", "manage subscript… |

## Hooks (auto-synced)

_119 hooks — `hooks/*.hook.ts`, grouped by dispatch event_

**FileChanged** (3):

`ClaudeMdIntegrity.hook.ts` · `PRDSync.hook.ts` · `SyncMemoryToDB.hook.ts`

**PostToolUse** (15):

`AlgorithmTracker.hook.ts` · `BehavioralAnomalyDetector.hook.ts` · `CatalogDispatch.hook.ts` · `CheckpointPerISC.hook.ts` · `ClaudeMdIntegrity.hook.ts` · `ContentInjectionScanner.hook.ts` · `ISAStubEnrichment.hook.ts` · `LoopDetector.hook.ts` · `OutputSecretsScanner.hook.ts` · `PRDSync.hook.ts` · `PlanningOracleFeedback.hook.ts` · `QuestionAnswered.hook.ts` · `ResearchCapture.hook.ts` · `SwitchboardSync.hook.ts` · `ToolError.hook.ts`

**PreToolUse** (12):

`AgentExecutionGuard.hook.ts` · `AgentStart.hook.ts` · `BudgetGuard.hook.ts` · `DeferRiskyOps.hook.ts` · `EgressClassGuard.hook.ts` · `LocalResearch.hook.ts` · `SecretInputScanner.hook.ts` · `SecurityValidator.hook.ts` · `SetQuestionTab.hook.ts` · `SkillGuard.hook.ts` · `SwitchboardPoll.hook.ts` · `UnicodeInjectionScanner.hook.ts`

**SessionEnd** (10):

`AssetRegistrySync.hook.ts` · `IntegrityCheck.hook.ts` · `MemoryCheckpoint.hook.ts` · `PromptHistorySync.hook.ts` · `SessionCleanup.hook.ts` · `SessionIndexUpdate.hook.ts` · `SessionSummary.hook.ts` · `UpdateCounts.hook.ts` · `WikilinkGraphSync.hook.ts` · `WorkCompletionLearning.hook.ts`

**SessionStart** (11):

`AbandonedStubReaper.hook.ts` · `CheckVersion.hook.ts` · `DebtSurface.hook.ts` · `HookHealer.hook.ts` · `IdentityPin.hook.ts` · `KittyEnvPersist.hook.ts` · `LiveContext.hook.ts` · `LoadContext.hook.ts` · `PromotePending.hook.ts` · `SelfHealingTripwire.hook.ts` · `StartupGreeting.hook.ts`

**Stop** (13):

`BudgetTracker.hook.ts` · `ClaimAttributionScan.hook.ts` · `DocIntegrity.hook.ts` · `FormatComplianceScan.hook.ts` · `GenerateHandoff.hook.ts` · `IdentityValidator.hook.ts` · `MemoryAppendOnStop.hook.ts` · `ResponseTabReset.hook.ts` · `SelfHealingTripwire.hook.ts` · `SessionLogCapture.hook.ts` · `SessionReport.hook.ts` · `SkillNudge.hook.ts` · `StopOrchestrator.hook.ts`

**StopFailure** (1):

`StopFailure.hook.ts`

**TaskCreated** (1):

`AlgorithmTracker.hook.ts`

**UserPromptSubmit** (15):

`AccountabilityCheck.hook.ts` · `AutoWorkCreation.hook.ts` · `CommitmentCheck.hook.ts` · `ComplexityRouter.hook.ts` · `ContextNudge.hook.ts` · `CouncilRouter.hook.ts` · `ExplicitRatingCapture.hook.ts` · `FormatReminder.hook.ts` · `ImplicitSentimentCapture.hook.ts` · `InboxCheck.hook.ts` · `InitiativeBinder.hook.ts` · `MemoryCheckpoint.hook.ts` · `ModeClassifier.hook.ts` · `PromptProcessing.hook.ts` · `SessionBusTail.hook.ts`

**(unwired / other)** (43):

`AgentEventVoice.hook.ts` · `AgentOutputCapture.hook.ts` · `AlgorithmDiscordDigest.hook.ts` · `AlgorithmDiscordThread.hook.ts` · `BackgroundAgentCleanup.hook.ts` · `BashOutputSanitizer.hook.ts` · `ConfigReload.hook.ts` · `ContainmentGuard.hook.ts` · `CwdChangedReanchor.hook.ts` · `DestructiveOpGuard.hook.ts` · `ElicitationGuard.hook.ts` · `FullPathGuard.hook.ts` · `ISASync.hook.ts` · `ITILDashboard.hook.ts` · `LastResponseCache.hook.ts` · `McpInspector.hook.ts` · `MemoPromotionExtractor.hook.ts` · `MemoryDashboard.hook.ts` · `MemoryRecallTelemetry.hook.ts` · `MemoryWriteHonestyGate.hook.ts` · `PendingPromotionNotice.hook.ts` · `PermissionDenied.hook.ts` · `PhaseTransitionGate.hook.ts` · `PostCompactRecovery.hook.ts` · `PreCompact.hook.ts` · `RatingCapture.hook.ts` · `RelationshipMemory.hook.ts` · `SchemaContradictionCheck.hook.ts` · `SessionAutoName.hook.ts` · `SessionBusSurface.hook.ts` · `SessionEnd.hook.ts` · `SessionStart.hook.ts` · `SessionTraceCapture.hook.ts` · `SmartApprover.hook.ts` · `SpineDocCheck.hook.ts` · `TaskAttribution.hook.ts` · `ToolActivityTracker.hook.ts` · `ToolSearchInjectionScan.hook.ts` · `UnifiedDashboard.hook.ts` · `UpdateTabTitle.hook.ts` · `UserPromptSubmit.hook.ts` · `VoiceCompletion.hook.ts` · `VoiceGate.hook.ts`

Updated: 2026-07-07T17:59:22.439Z

<!-- AUTOGEN:catalogue END -->

## Engine Presence (auto-synced)

- PNC: status=ended, agent=PAI Nova, last_seen=2026-07-07T18:05:00.099Z, session=2d823abf-75bd-4f70-ae81-975b4236435b, host=pai-primary, model=unknown
- PNG: status=ended, agent=PAI Nova, last_seen=2026-07-04T11:42:42.041Z, session=png-shell-1783165361-0, host=pai-primary, model=unknown
- PNK: status=ended, agent=PAI Nova, last_seen=2026-07-07T18:38:08.696Z, session=ses_0c5536aa3fferXjHBqiyM4HHOW, host=pai-primary, model=unknown
- PNX: status=ended, agent=PAI Nova, last_seen=2026-07-07T18:00:32.576Z, session=pnx-shim-1783447232-0, host=pai-primary, model=unknown
- UNKNOWN: status=ended, agent=PAI Nova, last_seen=2026-07-07T18:03:21.143Z, session=ses_0c243cab2ffeRS7y53TPx4NOPQ, host=pai-primary, model=unknown

Updated: 2026-07-07T18:38:08.697Z
