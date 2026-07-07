# PAI Quick Start Guide

Get PAI running in under 10 minutes. This fast track assumes you're comfortable with Linux/macOS terminal and have basic dev experience.

## Prerequisites Check

Before you begin, verify you have:

```bash
# Required
node --version      # v18+ or v20+
bun --version       # Latest (or install below)
git --version       # Any recent version

# Optional but recommended
docker --version    # For Ollama local models
tmux --version      # For session management
```

## One-Command Install

```bash
curl -fsSL https://raw.githubusercontent.com/yourusername/PAI/main/install.sh | bash
```

**What this does:**
1. Installs Bun if missing
2. Clones PAI to `~/.claude/`
3. Creates required directories
4. Sets up basic configuration
5. Installs dependencies
6. Runs initial setup wizard

## Manual Install (5 Steps)

### 1. Install Bun

```bash
curl -fsSL https://bun.sh/install | bash
```

### 2. Clone and Setup

```bash
# Clone repository
git clone https://github.com/yourusername/PAI.git ~/.claude
cd ~/.claude

# Install dependencies
bun install

# Create required directories
mkdir -p ~/MEMORY/{WORK,LEARNING,STATE,RELATIONSHIP,RESEARCH}
mkdir -p ~/.config/PAI
```

### 3. Configuration Wizard

```bash
bun PAI/Tools/setup-wizard.ts
```

The wizard will ask for:
- Your name and preferences
- OpenAI/Anthropic API keys (optional)
- Notification preferences
- Voice settings

### 4. Build Core Files

```bash
# Generate CLAUDE.md from template
bun PAI/Tools/BuildCLAUDE.ts

# Verify setup
bun PAI/Tools/HealthCheck.ts
```

### 5. First Run

```bash
# Start Claude Code
claude

# Or if using the CLI
claude-code --project ~/.claude
```

## Essential Configuration

### API Keys

Create `~/.config/PAI/secrets.env`:

```bash
# Required for full functionality
ANTHROPIC_API_KEY=sk-ant-xxxxx

# Optional - for multi-model routing
OPENAI_API_KEY=sk-xxxxx
GEMINI_API_KEY=xxxxx

# Optional - for notifications
ELEVENLABS_API_KEY=xxxxx
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/xxxxx
```

### settings.json Essentials

Edit `~/.claude/settings.json`:

```json
{
  "principal": {
    "name": "YourName",
    "location": "Your City"
  },
  "daidentity": {
    "name": "Nova",
    "displayName": "NOVA"
  },
  "env": {
    "PAI_DIR": "/home/youruser/.claude",
    "MEMORY_DIR": "/home/youruser/MEMORY"
  }
}
```

## Verify Installation

```bash
# Run health check
bun ~/.claude/PAI/Tools/HealthCheck.ts

# Expected output:
# ✓ Bun runtime
# ✓ Required directories
# ✓ settings.json valid
# ✓ CLAUDE.md generated
# ✓ Skills loaded (120+)
```

## First Tasks

Try these to verify everything works:

```bash
# In Claude Code interface
/help                    # See available commands
/work                    # List recent work sessions
```

**Create your first task:**

```
Hi Nova, help me understand how PAI works
```

The Algorithm should engage and walk you through the system.

## Optional: Local Models (Ollama)

For cost-free local inference:

```bash
# Install Ollama
curl -fsSL https://ollama.ai/install.sh | sh

# Pull recommended models
ollama pull llama3.2:3b
ollama pull gemma2:9b
ollama pull qwen2.5:7b

# Start Ollama service
systemctl --user enable --now ollama
```

Update `settings.json`:

```json
{
  "env": {
    "OLLAMA_HOST": "http://localhost:11434",
    "PAI_OLLAMA_FAILOVER": "true"
  }
}
```

## Optional: Voice Notifications

```bash
# Start voice server
cd ~/.claude/PAI/Tools
bun VoiceServer.ts &

# Or install as systemd service
cp systemd/pai-voice-server.service ~/.config/systemd/user/
systemctl --user enable --now pai-voice-server
```

## Common Quick-Start Issues

### "Command not found: bun"
```bash
# Add to your shell rc file (~/.bashrc or ~/.zshrc)
export PATH="$HOME/.bun/bin:$PATH"
source ~/.bashrc
```

### "Permission denied" errors
```bash
# Fix ownership
sudo chown -R $USER:$USER ~/.claude ~/MEMORY
chmod -R 755 ~/.claude/PAI/Tools
```

### "Module not found" errors
```bash
cd ~/.claude
bun install --force
```

## Next Steps

- **Full setup**: Read [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md)
- **Architecture**: See [ARCHITECTURE.md](ARCHITECTURE.md)
- **Configuration**: Check [CONFIGURATION.md](CONFIGURATION.md)
- **Skills**: Explore `~/.claude/skills/` directory
- **Algorithm**: Read `PAI/Algorithm/LATEST` for current version

## Getting Help

- Documentation: `~/.claude/PAI/DOCUMENTATION/`
- Issues: GitHub Issues
- Community: Discord server
- Ask PAI: Just ask Nova directly in Claude Code!

---

**You're ready!** PAI is now installed. The real learning happens through use — start with simple tasks and let PAI guide you through its capabilities.
