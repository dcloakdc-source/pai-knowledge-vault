# PAI One-Command Installer

Production-ready installer for Personal AI Infrastructure (PAI). Install PAI with a single command on Ubuntu, Debian, or macOS.

## Quick Start

```bash
curl -fsSL https://raw.githubusercontent.com/duanecollins/PAI/main/install.sh | bash
```

Or download and inspect first:

```bash
curl -fsSL https://raw.githubusercontent.com/duanecollins/PAI/main/install.sh -o install.sh
less install.sh
bash install.sh
```

## What It Does

The installer automates the complete PAI setup process:

1. **Detects OS** - Ubuntu/Debian/macOS
2. **Checks prerequisites** - Git, curl, etc.
3. **Installs Bun** - Primary JavaScript runtime
4. **Installs dependencies** - yt-dlp, tmux (optional)
5. **Creates directory structure** - MEMORY/, KNOWLEDGE/, WORK/
6. **Clones PAI repository** - Latest from GitHub
7. **Installs PAI dependencies** - `bun install`
8. **Creates config files** - settings.json, CLAUDE.md
9. **Installs systemd services** - Optional background services
10. **Runs health check** - Validates installation
11. **Prints next steps** - How to configure and use PAI

## Supported Platforms

| Platform | Version | Status |
|----------|---------|--------|
| Ubuntu | 22.04 LTS | ✅ Fully tested |
| Ubuntu | 24.04 LTS | ✅ Fully tested |
| Debian | 12+ | ✅ Supported |
| macOS | 13+ | ✅ Supported |

## Prerequisites

### Minimum Requirements

- **OS**: Ubuntu 22.04+, Debian 12+, or macOS 13+
- **CPU**: 2 cores
- **RAM**: 4GB
- **Storage**: 20GB available
- **Network**: Reliable internet connection

### Auto-Installed

The installer will automatically install:

- **Bun** - JavaScript runtime (required)
- **Git** - Version control (required)
- **curl** - HTTP client (required)
- **yt-dlp** - YouTube downloader (optional)
- **tmux** - Terminal multiplexer (optional)

### Not Included (Optional)

These can be installed separately after PAI installation:

- **Ollama** - Local LLM inference
  ```bash
  curl -fsSL https://ollama.com/install.sh | sh
  ```

## Installation Steps

### 1. Run Installer

```bash
bash ~/.claude/PAI/install.sh
```

The installer will:
- Display OS detection
- Check prerequisites
- Install missing dependencies
- Create directory structure
- Clone PAI repository
- Install PAI dependencies
- Run health check

### 2. Configure Settings

Edit your settings file:

```bash
nano ~/.claude/settings.json
```

Key settings to configure:

```json
{
  "principal": {
    "name": "Your Name",
    "email": "your@email.com"
  },
  "daidentity": {
    "name": "Nova",
    "displayName": "NOVA"
  }
}
```

### 3. Add API Keys

Add your Anthropic API key:

```bash
export ANTHROPIC_API_KEY='sk-ant-...'
```

Make it permanent:

```bash
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.bashrc
source ~/.bashrc
```

Optional API keys:

```bash
export OPENAI_API_KEY='sk-...'          # For OpenAI models
export GOOGLE_API_KEY='AIza...'         # For Gemini models
export ELEVENLABS_API_KEY='...'         # For voice synthesis
```

### 4. Test Installation

```bash
cd ~/.claude/PAI
bun test
```

### 5. (Optional) Install Local Models

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Pull recommended models
ollama pull llama3.2
ollama pull gemma2:9b
ollama pull qwen2.5:7b
```

### 6. Start Using PAI

Open Claude Code and start working!

## Directory Structure

After installation, you'll have:

```
~/.claude/
├── PAI/                    # PAI repository
│   ├── Algorithm/          # Core algorithm
│   ├── Tools/             # CLI tools
│   ├── skills/            # Skill system
│   ├── DOCUMENTATION/     # Full docs
│   └── install.sh         # This installer
├── MEMORY/                # Persistent memory
│   ├── auto/              # Auto-saved context
│   ├── Research/          # Research outputs
│   ├── SESSIONS/          # Session logs
│   ├── STATE/             # State files
│   └── WORK/              # Work sessions
├── KNOWLEDGE/             # Knowledge base
│   ├── People/            # Person notes
│   ├── Companies/         # Company notes
│   ├── Ideas/             # Idea notes
│   └── Research/          # Research notes
├── WORK/                  # Active work
├── downloads/             # Downloads
├── backups/               # Backups
├── settings.json          # User settings
└── CLAUDE.md             # System prompt
```

## Systemd Services (Linux Only)

The installer can optionally set up systemd user services:

```bash
# During installation, answer 'y' when prompted:
Install systemd services? (y/N) y
```

Available services:

- `pai-engine-bench-smoke.service` - Quick engine smoke tests
- `pai-engine-bench-weekly.service` - Weekly engine benchmarks
- `upgrade-check.service` - Check for PAI updates
- `opencode-exporter.service` - Export OpenCode sessions

Start a service:

```bash
systemctl --user start pai-engine-bench-smoke
```

Enable at boot:

```bash
systemctl --user enable pai-engine-bench-smoke
```

Check status:

```bash
systemctl --user status pai-engine-bench-smoke
```

## Health Check

The installer runs a comprehensive health check:

```
[SUCCESS] ✓ Bun: 1.3.13
[SUCCESS] ✓ Git: 2.43.0
[SUCCESS] ✓ PAI repository
[SUCCESS] ✓ Dependencies installed
[SUCCESS] ✓ yt-dlp (optional)
[SUCCESS] ✓ tmux (optional)
[INFO]    Ollama not installed (optional - for local models)

Health check: 4 passed, 0 failed
```

You can re-run the health check anytime:

```bash
# Manual health check
bun ~/.claude/PAI/Tools/SystemHealth.ts
```

## Troubleshooting

### Bun Installation Fails

```bash
# Manual Bun installation
curl -fsSL https://bun.sh/install | bash

# Add to PATH
echo 'export PATH="$HOME/.bun/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Permission Denied

```bash
# Make installer executable
chmod +x install.sh
bash install.sh
```

### Repository Clone Fails

```bash
# Clone manually
git clone https://github.com/duanecollins/PAI.git ~/.claude/PAI
cd ~/.claude/PAI
bun install
```

### Dependencies Fail to Install

```bash
# Clean and reinstall
cd ~/.claude/PAI
rm -rf node_modules bun.lockb
bun install
```

### Health Check Fails

Check specific components:

```bash
# Check Bun
bun --version

# Check Git
git --version

# Check directory structure
ls -la ~/.claude/

# Check PAI repository
ls -la ~/.claude/PAI/
```

## Updating PAI

Pull latest changes:

```bash
cd ~/.claude/PAI
git pull
bun install
```

Or re-run the installer:

```bash
bash ~/.claude/PAI/install.sh
# Answer 'y' to update existing installation
```

## Uninstalling

To remove PAI:

```bash
# Stop services (if installed)
systemctl --user stop pai-*.service
systemctl --user disable pai-*.service

# Remove PAI directory
rm -rf ~/.claude/PAI

# Optional: Remove all PAI data
rm -rf ~/.claude/
```

Keep your data and just remove the code:

```bash
# Remove only the PAI repository
rm -rf ~/.claude/PAI

# Keep MEMORY/, KNOWLEDGE/, WORK/, settings.json
```

## Testing the Installer

Run the test suite:

```bash
bash ~/.claude/PAI/test-install.sh
```

Expected output:

```
PAI Installation Test Suite
═══════════════════════════════════════════════════════════
[TEST] Installer exists... PASS
[TEST] Installer is executable... PASS
[TEST] Has error handling... PASS
[TEST] Has OS detection... PASS
[TEST] Has Bun installation... PASS
[TEST] Has dependency installation... PASS
[TEST] Has health check... PASS
[TEST] Has directory creation... PASS
[TEST] Has version checks... PASS
[TEST] Has next steps... PASS

Passed: 10  Failed: 0

✓ All tests passed!
```

## Next Steps

After successful installation:

1. **Read the docs**:
   ```bash
   less ~/.claude/PAI/DOCUMENTATION/QUICKSTART.md
   less ~/.claude/PAI/DOCUMENTATION/DEPLOYMENT_GUIDE.md
   ```

2. **Configure your setup**:
   ```bash
   nano ~/.claude/settings.json
   ```

3. **Add API keys**:
   ```bash
   export ANTHROPIC_API_KEY='your-key'
   ```

4. **Test PAI**:
   ```bash
   cd ~/.claude/PAI
   bun test
   ```

5. **Start using Claude Code** with PAI!

## Support

- **Documentation**: `~/.claude/PAI/DOCUMENTATION/`
- **Issues**: GitHub Issues
- **Discussions**: GitHub Discussions

## License

See LICENSE file in the PAI repository.

---

**Installation complete!** 🎉

You now have a fully functional Personal AI Infrastructure. Start using Claude Code to interact with PAI.
