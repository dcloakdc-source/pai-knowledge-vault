# PAI NAS Port Registry
Location: `\\192.168.50.6\pai\PORTS.md`

## Active Services

| Service | Port(s) | Protocol | Description | Source |
|---------|---------|----------|-------------|--------|
| **pai-code-server** | 8444 | HTTPS | Web IDE | `docker-compose.qnap.yml` |
| **Wazuh Dashboard** | 443/8443? | HTTPS | SIEM Web Interface | `docker-compose.actions.yml` |
| **Wazuh API** | 55000 | TCP | Manager API | `docker-compose.actions.yml` |
| **Wazuh Agent** | 1514 | TCP/UDP | Agent Registration/Data | `docker-compose.actions.yml` |
| **Wazuh Indexer** | 9200 | HTTP | OpenSearch Database | `docker-compose.actions.yml` |
| **Neo4j** | 7474 | HTTP | Knowledge Graph Browser | `madeinoz-knowledge-system` |
| **Neo4j Bolt** | 7687 | TCP | Knowledge Graph Driver | `madeinoz-knowledge-system` |
| **Knowledge MCP** | 9500 | HTTP | Graphiti MCP Server | `madeinoz-knowledge-system` |

## Discovered Services (Network Scan 2026-03-06)

| Host | Service | Port(s) | Notes | Status |
|------|---------|---------|-------|--------|
| 172.30.1.3 | PAI Unified Command Hub | 8080 | Full dashboard web app (Outfit/dark theme). **Needs auth + TLS.** | ⚠️ Unprotected |
| 192.168.50.117 | Knowledge MCP host | 22, 9500 | Ubuntu — `OpenSSH_10.0p2 Ubuntu`, `Python/3.13 aiohttp/3.11.16`. Separate host from NAS. | Register in swarm-nodes |
| 192.168.50.10 | Ubuntu Linux server | 22 | `OpenSSH_10.0p2 Ubuntu-5ubuntu5`. Unknown purpose. | Investigate |
| 192.168.50.7 | NAS secondary NIC | 22, 80, 139, 445, 8080, 8443, 8444 | QNAP TS-464 second network interface (.6 is primary). Same device. | ✅ Identified |
| 192.168.50.5 | Windows Workstation | 22, 139, 445, 8443 | `OpenSSH_for_Windows_10.0`. Antigravity SSH client. Ollama at :11434. | User machine |
| 192.168.50.1 | Router/Gateway | 22, 53, 139, 445, 8443 | Dropbear SSH + httpd/3.0. SMB exposed — review. | Router |
| 192.168.50.115 | Brother MFC-L3710CW | 80, 443 | Network printer. `debut/1.30`. Enable admin password. | Printer |
| 192.168.50.212 | Marvell-WM (WiFi AP/IoT) | 80, 443 | Likely WiFi AP or smart device. Move to IoT VLAN. | IoT — isolate |
| 192.168.50.8 | SMB Host | 139, 445, 8080, 8443 | Unknown device. SMB exposed. | Investigate |
| 192.168.50.9 | SMB Host | 139, 445, 8080, 8443 | Unknown device. SMB exposed. | Investigate |

## PAI Node Services (192.168.50.10 — ThinkCentre M710q)

| Service | Port | Description | Status |
|---------|------|-------------|--------|
| **pai-command-center** | 8766 | PAI Dashboard | ✅ Running |
| **pai-functions** | 8890 | Functions API | ✅ Running |
| **pai-voice-server** | 8888 | ElevenLabs TTS | ✅ Running |
| **pai-dtrvmt** | 8000 | DTRVMT FastAPI backend (SQLite) | ✅ Running (2026-03-17) |
| **AIDA backend** | 8000* | AI-Driven Security Assessment (needs PostgreSQL/Docker) | ⏳ Pending |

*AIDA shares port 8000 — will need port coordination once deployed.

## Reserved / Conflicts
- **8000**: Frequently used default (fastapi, python http). Avoid.
- **8080**: QNAP Default Web UI / PAI Command Hub (172.30.1.3)
- **8090**: QNAP Default?

## Guidelines
- Always update this file when deploying a new service.
- Use `9xxx` range for PAI-specific internal tools to avoid QNAP conflicts.
- Check `netstat -tunlp` or `docker ps` on NAS before assigning.
- *Last updated: 2026-03-06 by Antigravity network scan*
