# SINA Master Architecture (SINA_ARCH.md)
*Source of Truth for Sovereign Identity & Network Assets*
*Created: 2026-04-15*

## 1. Identity Registry (WIMSE / RBAC)
Combined from `RBAC_SCHEMA.md` and `IAM_ARCHITECTURE.md`.

| UID | Name | Role / Tier | WIMSE URI |
|-----|------|-------------|-----------|
| `USR-001` | Duane | Sovereign (T0) | `spiffe://pai.local/user/duane` |
| `AGT-001` | PAI Nova Claude (PNC) | Sovereign (T0) | `spiffe://pai.local/agent/pai-nova-claude` |
| `AGT-002` | PAI Nova Antigravity CLI (PNG) | Privileged (T1) | `spiffe://pai.local/agent/pai-nova-antigravity-cli` |
| `AGT-003` | PAI Nova OpenCode (opai) | Privileged (T1) | `spiffe://pai.local/agent/pai-nova-opencode` |
| `AGT-004` | PAI MCP Server | Standard (T2) | `spiffe://pai.local/agent/mcp-server` |
| `AGT-005` | PAI Antigravity IDE | Privileged (T1) | `spiffe://pai.local/agent/antigravity-ide` |

## 2. Network Infrastructure
Summarized from `SystemManifest.json`.

### Hypervisors
- **Proxmox (prox)**: 192.168.50.5 (LAN) / 100.127.249.19 (Tailscale)
  - VM 100: pai-primary (192.168.50.20)
  - VM 125: desktop-2tg5hm1 (Stopped - Recovery)
  - VM 130: haos17.1 (192.168.50.131?)

### Gateways & Mesh
- **Primary Router**: 192.168.50.1 (Asus ROG GT-AX11000 Pro)
- **Mesh Nodes**: 192.168.50.2, 192.168.50.3 (TP-Link Deco)

### Storage (QNAP)
- **qnap-nas**: 192.168.50.6
- **NAS6DB92A**: 192.168.50.7
- **NASE8C857**: 192.168.50.9

## 3. Governance
- **Secret Strategy**: Vaultwarden + `fetch-secret.sh` proxy.
- **JIT Flow**: 60-min TTL, single-use tokens in `MEMORY/STATE/jit-tokens/`.
- **Coordination**: PoW-signed inter-agent inbox messages.
- **Resumable SSH**: SSH `ControlMaster` enabled globally via `~/.ssh/sockets/`. `Mosh` prioritized for network-layer roaming (UDP 60000-61000).

## 4. Pending Discovery (Gaps)
- [ ] Reconcile non-responsive IPs (likely sleeping IoT):
  - 192.168.50.11, .248
- [x] Verify SSH access for all T1 agents to their respective domains.
- [x] Complete audit of `~/.ssh/config` (Created 2026-04-15).
