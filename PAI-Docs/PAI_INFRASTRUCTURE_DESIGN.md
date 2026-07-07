# PAI Infrastructure Design — Hardware Optimization & Resilience
**Version:** 1.0  
**Date:** 2026-04-14  
**Audited by:** PNC (PAI Nova Claude)

---

## 1. Hardware Inventory & Capability Weighting

### 1.1 Node Registry

| Node | LAN IP | Tailscale | CPU | RAM | GPU | Storage | Status |
|---|---|---|---|---|---|---|---|
| **pai-primary** | 192.168.50.20 | 100.71.20.100 | i7-11800H (8C) | 32GB | RTX 3060 Mobile 6GB VRAM | 292GB SSD | ✅ Online |
| **pai-dashboard / QNAP NAS** | 192.168.50.6 | 100.123.24.31 | ARM/QNAP | ~4GB | None | Multi-TB HDD | ✅ Online |
| **M710q (Authentik)** | 192.168.50.10/.28 | 100.124.247.23 | i5-7400T (4C) est. | 16-32GB est. | None | SSD est. | ⚠️ UFW-Firewalled |
| **prox (Proxmox)** | 192.168.50.5 | 100.127.249.19 | Unknown | Unknown | None | WD Black 1TB | ✅ Online |
| **Windows WS 1** | 192.168.50.5 | 100.71.235.13 | Unknown | Unknown | Unknown | Unknown | ⚠️ No services |
| **Windows WS 2** | — | 100.77.80.47 | Unknown | Unknown | Unknown | Unknown | ⚠️ No services |
| **Legion Y530** | — | 100.99.240.58 | i7-8750H est. | 16GB est. | GTX 1060 6GB | SSD | ⚠️ Unknown |
| **ThinkPad T470** | — | 100.101.117.108 | i5-7300U (2C) | 8-16GB | None | SSD | ⚠️ Unknown |

### 1.2 Capability Weights (Scoring: 1–10 per capability)

| Node | LLM Inference | API Services | Storage/NFS | IAM/Security | Workflow | Backup Target | Notes |
|---|---|---|---|---|---|---|---|
| **pai-primary** | **10** | **9** | 3 | 5 | 8 | 2 | Primary; RTX 3060 for GPU inference |
| **QNAP NAS** | 0 | 2 | **10** | 1 | 1 | **10** | Tailscale Funnel; NFS authority |
| **M710q** | 4 | 7 | 3 | **10** | 5 | 3 | Authentik host; needs UFW opened |
| **prox** | 5 | 8 | 6 | 4 | **9** | 6 | VM isolation; currently offline |
| **Legion Y530** | 7 | 4 | 1 | 1 | 3 | 1 | GTX 1060 secondary inference |
| **ThinkPad T470** | 1 | 5 | 1 | 3 | 4 | 2 | Lightweight microservices |

---

## 2. Current Service Inventory

### 2.1 All Services on pai-primary (Single Point of Failure)

| Service | Port | Process | RAM Est. | Criticality | Notes |
|---|---|---|---|---|---|
| **Ollama** | 11434 | Go binary | 2-4GB (models) | **Critical** | GPU inference; 7 models loaded |
| **CommandCenter** | 8766 | Bun | ~200MB | **Critical** | Dashboard; API hub; JWT auth |
| **FunctionsAPI** | 8890 | Bun | ~200MB | **Critical** | Credential gateway; 27+ endpoints |
| **Voice Server** | 8888 | Bun | ~50MB | High | ElevenLabs TTS proxy; phase announcements |
| **Memory MCP SSE** | 8891 | Bun | ~100MB | High | ICM / Infinite Context Memory |
| **Ollama A2A Bridge** | 8892 | Bun | ~50MB | High | Agent-to-Ollama routing |
| **Mastra** | 4111 | Bun | ~300MB | Medium | Workflow runtime; harvest-distill pipeline |
| **Inbox Watchdog** | — | Bun | ~30MB | High | Agent inbox fs.watch; event trigger |
| **Switchboard Bridge** | — | Python | ~50MB | High | Inter-agent messaging (SwitchboardBridge.py) |
| **Redis** | 6379 | redis-server | ~30MB | Medium | Cache/queue |
| **Litestream** | — | Go binary | ~20MB | High | memory.db → QNAP NFS replication |
| **Hyperledger Identus** | — | Java 21 | ~490MB | Medium | Sovereign DID cloud agent |
| **Atala PRISM Node** | — | Java 11 | ~490MB | Medium | DID ledger node (PRISM protocol) |

**Total RAM consumption estimate: ~8-11GB active** against 32GB physical + 8GB swap (5.8GB in use).  
⚠️ The two Java processes (Identus + PRISM Node) account for ~1GB RAM combined and represent the biggest inefficiency on pai-primary.

### 2.2 Utilization Assessment

| Resource | Current | Capacity | Headroom | Risk |
|---|---|---|---|---|
| RAM | ~10GB used + 5.8GB swap | 32GB | ~16GB free RAM | 🟡 Swap in use; Java bloat |
| CPU | Low (idle ~3%) | 8C | High | 🟢 |
| GPU VRAM | ~4-6GB Ollama | 6GB | ~0-2GB | 🟡 Models compete for VRAM |
| SSD | 103GB used | 292GB | 189GB | 🟢 |
| NFS (QNAP) | backup only | Multi-TB | Abundant | 🟢 |

---

## 3. Recommended Service Placement

### 3.1 Placement Philosophy

1. **pai-primary**: GPU inference + latency-critical APIs. Keep lean.
2. **M710q**: Security perimeter (Authentik), heavyweight JVM services, secondary compute.
3. **QNAP NAS**: Persistent state, backups, public-facing endpoints via Tailscale Funnel.
4. **Proxmox** (when online): VM-isolated redundancy for Tier 1 services.
5. **Legion Y530**: Secondary Ollama inference (failover + parallel).

### 3.2 Recommended Target State

| Service | Current Host | Recommended Host | Reason |
|---|---|---|---|
| Ollama (primary) | pai-primary | pai-primary ✅ | RTX 3060 Mobile; GPU inference |
| Ollama (secondary/failover) | None | **Legion Y530** | GTX 1060 6GB; failover + parallel |
| CommandCenter | pai-primary | pai-primary ✅ | Latency-critical; Tailscale-accessible |
| FunctionsAPI | pai-primary | pai-primary ✅ | Core credential gateway |
| Voice Server | pai-primary | pai-primary ✅ | Low overhead; keep local |
| Memory MCP SSE | pai-primary | pai-primary ✅ | db co-located with litestream source |
| Ollama A2A Bridge | pai-primary | pai-primary ✅ | Must be co-located with Ollama |
| Mastra | pai-primary | **M710q or prox** | Not latency-critical; frees pai-primary RAM |
| Inbox Watchdog | pai-primary | pai-primary ✅ | Watches local filesystem |
| Switchboard Bridge | pai-primary | pai-primary ✅ | Requires MEMORY/STATE access |
| Redis | pai-primary | **M710q** | Shared cache; offload RAM |
| Litestream | pai-primary | pai-primary ✅ | Source DB must stay co-located |
| **Hyperledger Identus** | pai-primary | **M710q** | ~490MB Java; not latency critical |
| **Atala PRISM Node** | pai-primary | **M710q** | ~490MB Java; identity infra |
| Authentik | M710q | M710q ✅ (fix UFW) | Already correct host; just needs firewall fix |
| Dashboard/Monitoring | QNAP | QNAP ✅ | Already on Tailscale Funnel |

**Net effect of migrations**: Remove ~1GB+ Java RAM from pai-primary, improve VRAM headroom for larger models.

---

## 4. Resilience Architecture

### 4.1 Failure Modes & Mitigations

| Failure | Impact | Current State | Mitigation |
|---|---|---|---|
| **pai-primary goes down** | ALL PAI services offline | No redundancy | Warm standby on Proxmox VM |
| **Ollama OOM / GPU crash** | All LLM inference stops | No fallback | Secondary Ollama on Legion Y530 |
| **memory.db corruption** | ICM memory lost | Litestream → QNAP | Add S3 off-site replica |
| **QNAP NFS mount fails** | Litestream can't write | No alert | Watchdog + alert on mount failure |
| **Authentik (M710q) goes down** | IAM stops issuing tokens | Effectively already down (UFW) | Fix UFW + health monitor |
| **Network partition (Tailscale)** | Remote agent access lost | No fallback | LAN-only mode; QNAP subnet router |
| **Redis goes down** | Cache/queue disruption | No persistence | Redis AOF + replication to M710q |

### 4.2 Tier Classification (by recovery priority)

#### Tier 1 — Recover within 60 seconds
- FunctionsAPI (8890) — credential gateway; blocks all agent operations
- CommandCenter (8766) — human control plane
- Ollama (11434) — primary inference

#### Tier 2 — Recover within 5 minutes
- Memory MCP SSE (8891) — ICM; agents degrade gracefully without it
- Voice Server (8888) — optional; algorithm phases go silent but continue
- Inbox Watchdog — agents retry inbox polling
- Switchboard Bridge — degrades to polling

#### Tier 3 — Recover within 30 minutes
- Mastra (4111) — workflows queue up
- A2A Bridge (8892) — routed through FunctionsAPI /v1/pai-pi instead
- Redis — rebuild cache from source

#### Tier 4 — Recover within 24 hours
- Hyperledger Identus / Atala PRISM Node — DID infra; infrequently used
- Authentik — IAM; agents use API keys as fallback

### 4.3 Near-Term Resilience Wins (Ordered by Impact/Effort)

| Priority | Action | Effort | Impact |
|---|---|---|---|
| 🔴 P1 | **Fix M710q UFW** — open ports 80/443/9000/9300 to 192.168.50.0/24 | Low | IAM comes online; Identus/PRISM can migrate |
| 🔴 P1 | **Add QNAP backup scope** — replicate MEMORY/STATE/ and PRD work dirs to NAS | Low | Protects all in-progress work |
| 🟡 P2 | **Migrate Java processes to M710q** — Identus + PRISM Node | Medium | Frees ~1GB RAM on pai-primary; reduces swap |
| 🟡 P2 | **Deploy secondary Ollama on Legion Y530** — configure FunctionsAPI failover | Medium | Ollama redundancy; load balancing |
| 🟡 P2 | **Add litestream S3 target** — Cloudflare R2 or Backblaze B2 | Low | Off-site DB backup |
| 🟠 P3 | **Bring Proxmox online** — diagnose SSH failure | Medium | VM isolation for services |
| 🟠 P3 | **Proxmox VM: warm standby** — replicate FunctionsAPI + CommandCenter config | High | Service-level redundancy |
| 🟠 P3 | **Mastra → M710q or prox VM** | Medium | Frees pai-primary RAM/CPU |
| 🟢 P4 | **Redis persistence + M710q replica** | Low | Cache durability |
| 🟢 P4 | **QNAP as secondary NFS for STATE backups** | Low | Broader backup coverage |

---

## 5. Ollama Model Optimization

### 5.1 Current Model Inventory (pai-primary)

| Model | Size | Use Case | Recommended Host |
|---|---|---|---|
| llama3.2:3b | ~2.0GB | LocalRunner (fast/classify) | pai-primary (fits in VRAM w/ others) |
| all-minilm:latest | ~46MB | Embeddings | pai-primary |
| mxbai-embed-large:latest | ~670MB | Embeddings | pai-primary |
| nemotron-mini:latest | ~2.7GB | Fast reasoning | pai-primary |
| deepseek-r1:7b | ~4.7GB | LocalReasoner | pai-primary primary / Legion failover |
| qwen2.5:7b | ~4.7GB | LocalAnalyst (code) | pai-primary primary / Legion failover |
| gemma2:9b | ~5.4GB | LocalSummarizer | ⚠️ Largest model; may evict others from VRAM |

**VRAM constraint**: RTX 3060 Mobile = 6GB VRAM.  
- gemma2:9b alone exceeds VRAM → falls to RAM → slow inference + swap pressure  
- Optimal: keep 3b/embed models always loaded; load 7b models on-demand; route gemma2:9b to Legion Y530 when available

### 5.2 VRAM-Aware Routing Recommendation

```
Priority 1: pai-primary GPU (RTX 3060 6GB)
  → llama3.2:3b, nemotron-mini, all models ≤5GB with headroom

Priority 2: pai-primary CPU RAM (when GPU full)
  → qwen2.5:7b, deepseek-r1:7b (fallback to slow RAM inference)

Priority 3: Legion Y530 (GTX 1060 6GB) — when configured
  → gemma2:9b (preferred), 7b models (parallel inference)
```

---

## 6. Data Resilience

### 6.1 Current Backup Coverage

| Data | Current Backup | Gap |
|---|---|---|
| memory.db (ICM) | Litestream → /mnt/pai/backup/ | No off-site |
| MEMORY/STATE/ | None | ❌ Agent state lost on failure |
| MEMORY/WORK/ (PRDs) | None | ❌ In-progress work lost |
| MEMORY/LEARNING/ | None | ❌ Algorithm reflections lost |
| ~/.claude/settings.json | None | ❌ Config lost |
| MEMORY/db/ | Litestream | ✅ (NAS only) |

### 6.2 Recommended Backup Strategy

```
Tier A — Continuous (Litestream):
  memory.db → QNAP NAS (current) + Cloudflare R2 (add)

Tier B — Hourly rsync to QNAP:
  MEMORY/STATE/
  MEMORY/WORK/
  MEMORY/LEARNING/
  PAI/ (config, manifests)

Tier C — Daily git push:
  ~/.claude/ repo → private remote (GitHub/Gitea)
  Already partially done via git history
```

---

## 7. Action Plan

### Phase 1 — Quick Wins (This Week)
1. **M710q UFW**: `ssh acp-admin@192.168.50.10 "sudo ufw allow from 192.168.50.0/24"` (then verify Authentik accessible on 9000/9300)
2. **Expand litestream scope**: Add MEMORY/STATE/ and MEMORY/WORK/ to litestream.yml with rsync replica to QNAP
3. **Add S3 off-site**: Configure litestream R2 or B2 target for memory.db

### Phase 2 — Service Migration (Next 2 Weeks)
1. Migrate Hyperledger Identus + Atala PRISM Node to M710q (once UFW fixed)
2. Deploy Ollama on Legion Y530 with pai-primary models; wire FunctionsAPI failover
3. Configure Redis persistence (AOF) + optional M710q replica

### Phase 3 — Full Resilience (After Proxmox Online)
1. Proxmox VM for warm standby: replicate FunctionsAPI + CommandCenter config
2. Move Mastra workflow runtime to Proxmox VM
3. Implement health watchdog with auto-failover routing in FunctionsAPI

---

*This document is the infrastructure source of truth. Update after each migration.*
