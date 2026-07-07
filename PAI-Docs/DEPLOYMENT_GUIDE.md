# PAI Deployment Guide

Comprehensive guide to deploying PAI (Personal AI Infrastructure) on your own hardware. This covers everything from hardware selection to production hardening.

## Table of Contents

- [Hardware Requirements](#hardware-requirements)
- [Software Prerequisites](#software-prerequisites)
- [Installation Methods](#installation-methods)
- [Core Configuration](#core-configuration)
- [Service Installation](#service-installation)
- [Security Hardening](#security-hardening)
- [Troubleshooting](#troubleshooting)
- [Maintenance](#maintenance)

## Hardware Requirements

### Minimum Viable Setup

Perfect for trying PAI on existing hardware:

- **CPU**: 2 cores (any modern x64 CPU)
- **RAM**: 4GB
- **Storage**: 20GB SSD
- **Network**: Reliable internet for API calls

**Capabilities with minimum:**
- Full Algorithm functionality
- Cloud-based models only (Anthropic, OpenAI)
- Basic skills and workflows
- Persistent memory up to ~50 sessions

### Recommended Setup

Best balance of performance and cost:

- **CPU**: 4+ cores (AMD Ryzen 5 or Intel i5 equivalent)
- **RAM**: 16GB
- **Storage**: 100GB SSD (NVMe preferred)
- **Network**: Gigabit ethernet
- **Optional**: GPU with 8GB+ VRAM for local models

**Capabilities with recommended:**
- All cloud features plus local inference
- Ollama models (llama3.2, gemma2, qwen2.5)
- Hundreds of sessions with full context
- Fast skill indexing and search
- Voice synthesis

### Production/Homelab Setup

For serious self-hosting and local-first operation:

- **CPU**: 8+ cores (AMD Ryzen 7/9 or Intel i7/i9)
- **RAM**: 32GB+ (64GB if running large local models)
- **Storage**: 
  - 256GB NVMe SSD (system and databases)
  - 1TB+ HDD/SSD (memory archives, backups)
- **GPU**: NVIDIA RTX 3060 12GB or better (for local LLMs)
- **Network**: Gigabit ethernet, static IP optional
- **Backup**: NAS or external drive for daily backups

**Example reference:** Duane's pai-primary runs on Proxmox with Ubuntu 24.04 VM, 8 vCPU, 32GB RAM, RTX 3060.

## Software Prerequisites

### Required

```bash
# Operating System
# - Ubuntu 22.04 LTS or 24.04 LTS (primary support)
# - macOS 13+ (tested but not primary)
# - Debian 12+ (community support)

# Runtime
bun >= 1.0.0           # Primary runtime

# Version Control
git >= 2.30            # For PAI updates and version management

# Terminal
bash >= 4.0            # Shell for scripts
tmux >= 3.0            # Session management (recommended)
```

### Optional but Recommended

```bash
# Local Models
docker >= 20.10        # For Ollama containerized deployment
ollama >= 0.1.0        # For local model inference

# Development
node >= 18.0           # Alternative runtime
python >= 3.10         # For Python-based tools

# Monitoring
htop                   # System monitoring
ncdu                   # Disk usage analysis
```

## Installation Methods

### Method 1: Automated Install (Recommended)

```bash
# One-line install
curl -fsSL https://raw.githubusercontent.com/yourusername/PAI/main/install.sh | bash

# Or download and inspect first
curl -fsSL https://raw.githubusercontent.com/yourusername/PAI/main/install.sh -o install.sh
less install.sh        # Review the script
bash install.sh
```

**What it does:**
1. Checks prerequisites and installs missing dependencies
2. Creates required directory structure
3. Clones PAI repository to `~/.claude/`
4. Runs `bun install` for dependencies
5. Executes setup wizard for initial configuration
6. Builds CLAUDE.md from template
7. Runs health check validation
8. Optionally installs systemd services

### Method 2: Manual Install

For full control over the installation process:

#### Step 1: Install Bun

```bash
# Linux/macOS
curl -fsSL https://bun.sh/install | bash

# Add to PATH
echo 'export PATH="$HOME/.bun/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Verify
bun --version
```

#### Step 2: Create Directory Structure

```bash
# PAI root
mkdir -p ~/.claude

# Memory directories
mkdir -p ~/MEMORY/WORK
mkdir -p ~/MEMORY/LEARNING
mkdir -p ~/MEMORY/STATE
mkdir -p ~/MEMORY/RELATIONSHIP
mkdir -p ~/MEMORY/RESEARCH
mkdir -p ~/MEMORY/WISDOM

# Configuration
mkdir -p ~/.config/PAI

# Optional: External project directory
mkdir -p ~/Projects
```

#### Step 3: Clone Repository

```bash
git clone https://github.com/yourusername/PAI.git ~/.claude
cd ~/.claude
```

#### Step 4: Install Dependencies

```bash
bun install
```

**What this installs:**
- TypeScript runtime dependencies
- MCP (Model Context Protocol) servers
- Tool dependencies (Qdrant client, notification libraries)
- Build tools

#### Step 5: Initial Configuration

```bash
# Copy template configuration
cp settings.json.template settings.json

# Create secrets file
touch ~/.config/PAI/secrets.env
chmod 600 ~/.config/PAI/secrets.env
```

Edit `~/.config/PAI/secrets.env`:

```bash
# API Keys (add the ones you'll use)
ANTHROPIC_API_KEY=sk-ant-xxxxx
OPENAI_API_KEY=sk-xxxxx
GEMINI_API_KEY=xxxxx

# Optional: Notifications
ELEVENLABS_API_KEY=xxxxx
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/xxxxx
NTFY_TOPIC=your-topic

# Optional: Services
QDRANT_API_KEY=xxxxx
FIRECRAWL_API_KEY=xxxxx
```

#### Step 6: Run Setup Wizard

```bash
bun PAI/Tools/setup-wizard.ts
```

The wizard configures:
- Your identity (name, location, preferences)
- AI assistant identity (name, voice)
- Model preferences
- Notification settings
- Optional services

#### Step 7: Build Core Files

```bash
# Generate CLAUDE.md from template
bun PAI/Tools/BuildCLAUDE.ts

# Verify
ls -la CLAUDE.md
```

#### Step 8: Validation

```bash
# Run health check
bun PAI/Tools/HealthCheck.ts

# Check all required components
bun PAI/Tools/SystemAudit.ts
```

Expected output:
```
✓ Runtime: Bun 1.x.x
✓ Directories: All required paths exist
✓ Configuration: settings.json valid
✓ CLAUDE.md: Generated successfully
✓ Skills: 120 loaded
✓ Hooks: 111 registered
✓ Memory: Accessible
```

## Core Configuration

### settings.json Explained

The `settings.json` file is your single source of truth. Key sections:

```json
{
  // Your identity
  "principal": {
    "name": "Your Name",
    "location": "Your City",
    "timezone": "America/New_York"
  },

  // AI assistant identity
  "daidentity": {
    "name": "Nova",
    "displayName": "NOVA",
    "voiceId": "af_sky"
  },

  // Files loaded at every session start
  "contextFiles": [
    "skills/PAI/SKILL.md",
    "skills/PAI/AISTEERINGRULES.md",
    "skills/PAI/USER/DAIDENTITY.md"
  ],

  // Environment variables
  "env": {
    "PAI_DIR": "/home/youruser/.claude",
    "MEMORY_DIR": "/home/youruser/MEMORY",
    "OLLAMA_HOST": "http://localhost:11434"
  },

  // Tool permissions
  "permissions": {
    "allow": [
      "Read(/home/youruser/.claude/**)",
      "Write(/home/youruser/.claude/MEMORY/**)",
      "Bash(git *)"
    ],
    "deny": [
      "Write(/etc/**)",
      "Bash(rm -rf *)"
    ]
  },

  // Lifecycle hooks
  "hooks": {
    "SessionStart": [
      {
        "command": "bun $PAI_DIR/hooks/SessionInit.hook.ts",
        "type": "command"
      }
    ]
  }
}
```

### User Context Setup

Create your personal context files in `PAI/USER/`:

```bash
cd ~/.claude/PAI/USER

# Your identity
nano ABOUTME.md

# AI steering rules (your preferences)
nano AISTEERINGRULES.md

# Life goals and missions (optional)
mkdir -p TELOS
nano TELOS/MISSIONS.md
```

### Algorithm Configuration

The Algorithm is versioned. Current version pointer:

```bash
cat ~/.claude/PAI/Algorithm/LATEST
# Outputs: v5.7.11 (or current version)

# View the active algorithm
cat ~/.claude/PAI/Algorithm/v5.7.11.md
```

## Service Installation

PAI includes several optional systemd services for enhanced functionality.

### Install All Services

```bash
cd ~/.claude/PAI/Tools/systemd
./install-services.sh
```

### Manual Service Installation

#### Voice Server

Provides TTS (text-to-speech) notifications:

```bash
# Copy service file
cp ~/.claude/PAI/Tools/systemd/pai-voice-server.service ~/.config/systemd/user/

# Edit paths if needed
nano ~/.config/systemd/user/pai-voice-server.service

# Enable and start
systemctl --user daemon-reload
systemctl --user enable --now pai-voice-server

# Verify
systemctl --user status pai-voice-server
curl http://localhost:8888/health
```

#### OmniPulse (Unified Daemon)

Central orchestration daemon for hooks, health checks, and dashboard:

```bash
cp ~/.claude/PAI/Tools/systemd/omnipulse.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now omnipulse

# Verify
curl http://localhost:31338/health
```

#### Memory MCP Server

Persistent memory across sessions:

```bash
cp ~/.claude/PAI/Tools/systemd/pai-memory.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now pai-memory

# Test
bun ~/.claude/PAI/Tools/Memory.ts status
```

### Optional: Ollama for Local Models

```bash
# Install Ollama
curl -fsSL https://ollama.ai/install.sh | sh

# Or via Docker
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama

# Pull recommended models
ollama pull llama3.2:3b      # Fast classification
ollama pull gemma2:9b        # Summarization
ollama pull qwen2.5:7b       # Structured extraction

# Verify
ollama list
curl http://localhost:11434/api/tags
```

## Security Hardening

### File Permissions

```bash
# Restrict access to PAI directory
chmod 700 ~/.claude
chmod 700 ~/MEMORY

# Secure secrets
chmod 600 ~/.config/PAI/secrets.env

# Verify
ls -la ~/.claude
ls -la ~/.config/PAI/
```

### API Key Rotation

```bash
# Use PAI's key rotation skill
bun ~/.claude/PAI/Tools/KeyRotation.ts

# Or manually
nano ~/.config/PAI/secrets.env
# Update keys, save

# Restart services
systemctl --user restart pai-*
```

### Network Security

If exposing services (not recommended for personal use):

```bash
# Firewall rules (example for UFW)
sudo ufw allow from 192.168.1.0/24 to any port 31338  # OmniPulse
sudo ufw deny 8888  # Voice server internal only

# Or use reverse proxy with auth
# See PAI/Tools/deploy/nginx-example.conf
```

### Permission Boundaries

Review and restrict tool permissions in `settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Read($PAI_DIR/**)",
      "Write($MEMORY_DIR/**)",
      "Bash(git * --dry-run)"
    ],
    "deny": [
      "Write(/etc/**)",
      "Write(/var/**)",
      "Bash(curl *)",  // If you want to prevent arbitrary downloads
      "Bash(rm -rf /)"
    ]
  }
}
```

## Troubleshooting

### Installation Issues

#### "bun: command not found"

```bash
# Check PATH
echo $PATH

# Add Bun to PATH
export PATH="$HOME/.bun/bin:$PATH"
echo 'export PATH="$HOME/.bun/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

#### "Permission denied" during install

```bash
# Fix ownership
sudo chown -R $USER:$USER ~/.claude ~/MEMORY ~/.config/PAI

# Fix permissions
chmod -R 755 ~/.claude
chmod 600 ~/.config/PAI/secrets.env
```

#### Dependencies fail to install

```bash
cd ~/.claude
rm -rf node_modules
bun install --force
```

### Runtime Issues

#### "Cannot find module" errors

```bash
# Rebuild
cd ~/.claude
bun install

# Verify
bun --bun run PAI/Tools/HealthCheck.ts
```

#### Services won't start

```bash
# Check service status
systemctl --user status pai-voice-server

# View logs
journalctl --user -u pai-voice-server -n 50

# Common fixes
systemctl --user daemon-reload
systemctl --user restart pai-voice-server
```

#### Ollama connection failures

```bash
# Check Ollama is running
curl http://localhost:11434/api/version

# If not running
systemctl start ollama  # System install
# OR
docker start ollama     # Docker install

# Verify host in settings.json
grep OLLAMA_HOST ~/.claude/settings.json
```

### Performance Issues

#### Slow startup

```bash
# Check file count in MEMORY
find ~/MEMORY -type f | wc -l

# If > 10,000 files, archive old sessions
bun ~/.claude/PAI/Tools/ArchiveWork.ts

# Reduce context files
# Edit settings.json → contextFiles (keep only essentials)
```

#### High memory usage

```bash
# Check memory
htop

# Limit context window in settings.json
# "env": {
#   "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "64000"  # Lower from 128000
# }
```

### See Also

- [TROUBLESHOOTING_METHOD.md](TROUBLESHOOTING_METHOD.md) - Systematic debugging approach
- [ARCHITECTURE.md](ARCHITECTURE.md) - How components interact
- GitHub Issues - Known issues and solutions

## Maintenance

### Regular Updates

```bash
# Update PAI
cd ~/.claude
git pull origin main
bun install

# Rebuild CLAUDE.md
bun PAI/Tools/BuildCLAUDE.ts

# Restart services
systemctl --user restart pai-*
```

### Backups

```bash
# Manual backup
bun ~/.claude/PAI/Tools/NasBackup.sh

# Or copy directly
rsync -av ~/.claude/ /backup/location/pai-backup/
rsync -av ~/MEMORY/ /backup/location/memory-backup/

# Automated daily backup (cron)
crontab -e
# Add: 0 2 * * * bun ~/.claude/PAI/Tools/NasBackup.sh
```

### Health Checks

```bash
# Run weekly
bun ~/.claude/PAI/Tools/HealthCheck.ts
bun ~/.claude/PAI/Tools/SystemAudit.ts

# Check disk usage
ncdu ~/MEMORY
```

### Cleanup

```bash
# Archive old work sessions (keeps last 100)
bun ~/.claude/PAI/Tools/ArchiveWork.ts

# Clean up Ollama models
ollama rm <unused-model>

# Prune old logs
journalctl --user --vacuum-time=30d
```

---

## Next Steps

- **Learn the system**: [ARCHITECTURE.md](ARCHITECTURE.md)
- **Configure deeply**: [CONFIGURATION.md](CONFIGURATION.md)
- **Start using**: Launch Claude Code and say "Hello Nova"
- **Explore skills**: `ls ~/.claude/skills/`
- **Read the Algorithm**: `cat ~/.claude/PAI/Algorithm/LATEST`

PAI is designed to grow with you. Start simple, add complexity as you understand the system.
