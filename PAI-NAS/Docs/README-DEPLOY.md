# PAI QNAP Deployment Package

## Quick Start

1. **Upload this entire folder** to your QNAP via File Station to `/share/PAI/`

2. **SSH to NAS** (or use QNAP Terminal in web UI):
   ```bash
   cd /share/PAI
   chmod +x scripts/start-pai.sh
   ./scripts/start-pai.sh core
   ```

3. **Access Dashboard**: http://192.168.50.52:8090

---

## Folder Structure After Upload

```
/share/PAI/
├── docker-compose.qnap.yml    # Main compose file
├── .env                        # Your API keys (edit this!)
├── scripts/
│   └── start-pai.sh           # Startup script
├── Dashboard/                  # API Bridge
├── Packs/                      # PAI Packs
├── data/                       # Persistent data (auto-created)
└── config/                     # Service configs
```

---

## Step-by-Step

### 1. Edit .env File

Before starting, edit `/share/PAI/.env` with your API keys:

```ini
GEMINI_API_KEY=your-key-here
OPENAI_API_KEY=your-key-here
ADMIN_PASSWORD=YourSecurePassword123!
```

### 2. Start Services

**Core only** (Dashboard):
```bash
./scripts/start-pai.sh core
```

**Full stack** (includes Wazuh SIEM + Identity):
```bash
./scripts/start-pai.sh all
```

### 3. Verify

| Service | URL |
|---------|-----|
| Dashboard | http://192.168.50.52:8090 |
| DTRVMT API | http://192.168.50.52:8000/docs |
| Wazuh SIEM | http://192.168.50.52:8601 |

---

## Commands

```bash
# Start all
./scripts/start-pai.sh all

# Start core only
./scripts/start-pai.sh core

# Start GRC only
./scripts/start-pai.sh grc

# Stop all
./scripts/start-pai.sh stop

# View logs
./scripts/start-pai.sh logs

# Check status
./scripts/start-pai.sh status
```
