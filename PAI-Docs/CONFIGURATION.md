# PAI Configuration Guide

Complete reference for configuring PAI. This covers every major configuration surface and how to customize PAI for your needs.

## Table of Contents

- [Configuration Files](#configuration-files)
- [settings.json Reference](#settingsjson-reference)
- [User Context](#user-context)
- [API Keys and Secrets](#api-keys-and-secrets)
- [Model Configuration](#model-configuration)
- [Skills](#skills)
- [Hooks](#hooks)
- [Memory](#memory)
- [Services](#services)
- [Advanced](#advanced)

## Configuration Files

PAI uses a layered configuration approach:

```
~/.claude/settings.json          # Main configuration (SSOT)
~/.config/PAI/secrets.env        # API keys and secrets
~/.claude/CLAUDE.md              # Generated from template
~/.claude/PAI/USER/              # Your personal context
~/.claude/PAI/Algorithm/LATEST   # Algorithm version pointer
```

**Load order:**
1. `settings.json` parsed at startup
2. Secrets loaded from `secrets.env`
3. `CLAUDE.md` loaded by Claude Code
4. User context loaded per `contextFiles`
5. Algorithm version from `LATEST` pointer

## settings.json Reference

The single source of truth. All configuration flows from here.

### Structure Overview

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "principal": { },          // Your identity
  "daidentity": { },         // AI assistant identity
  "contextFiles": [ ],       // Force-loaded files
  "env": { },                // Environment variables
  "permissions": { },        // Tool permissions
  "hooks": { },              // Lifecycle hooks
  "counts": { },             // Statistics (auto-updated)
  "enabledPlugins": { },     // MCP plugins
  "effortLevel": "high"      // Default effort
}
```

### Principal (Your Identity)

```json
{
  "principal": {
    "name": "Your Name",
    "shortName": "YourName",
    "location": "Your City, State",
    "timezone": "America/New_York",
    "email": "you@example.com"
  }
}
```

**Used for:**
- Personal pronouns in responses
- Timezone for scheduling and timestamps
- Email for notifications (if configured)

### DA Identity (AI Assistant)

```json
{
  "daidentity": {
    "name": "Nova",
    "displayName": "NOVA",
    "fullName": "Nova — Personal AI",
    "color": "#3B82F6",
    "voiceId": "af_sky",
    "mainDAVoiceID": "af_sky",
    
    "voice": {
      "speed": 1.1,
      "stability": 0.35,
      "similarity_boost": 0.8,
      "style": 0.9,
      "use_speaker_boost": true,
      "volume": 0.8
    },
    
    "voices": {
      "main": {
        "voiceId": "kokoro:af_sky",
        "speed": 1.1,
        "stability": 0.35
      },
      "algorithm": {
        "voiceId": "kokoro:af_sky",
        "speed": 1.1
      },
      "ollama": {
        "voiceId": "am_michael",
        "speed": 1.0
      }
    }
  }
}
```

**Voice IDs:**
- `kokoro:af_sky` - Female, warm, professional (default)
- `kokoro:af_bella` - Female, clear, energetic
- `am_adam` - Male, calm, authoritative
- `am_michael` - Male, neutral, steady

**Voice parameters:**
- `speed`: 0.5-2.0 (1.0 = normal)
- `stability`: 0-1 (higher = more consistent)
- `similarity_boost`: 0-1 (higher = closer to reference)
- `style`: 0-1 (exaggeration level)

### Context Files

Files force-loaded at every session start:

```json
{
  "contextFiles": [
    "skills/PAI/SKILL.md",
    "skills/PAI/AISTEERINGRULES.md",
    "skills/PAI/USER/AISTEERINGRULES.md",
    "skills/PAI/USER/DAIDENTITY.md"
  ]
}
```

**Best practices:**
- Keep to essentials (3-5 files)
- Each file should be <500 lines
- High-frequency context only
- Use routing table for on-demand loading

**Anti-patterns:**
- Loading entire skill catalog
- Loading project-specific docs (use dynamic loading)
- Redundant content across files

### Environment Variables

```json
{
  "env": {
    "PAI_DIR": "/home/user/.claude",
    "MEMORY_DIR": "/home/user/MEMORY",
    "PROJECTS_DIR": "/PAI/",
    
    "ANTHROPIC_SMALL_FAST_MODEL": "claude-haiku-4-5",
    "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "128000",
    
    "OLLAMA_HOST": "http://192.168.50.20:11434",
    "PAI_OLLAMA_FAILOVER": "true",
    "PAI_OLLAMA_FAILOVER_MODEL": "ollama/qwen2.5:7b",
    
    "BASH_DEFAULT_TIMEOUT_MS": "600000",
    "EDITOR": "nano",
    "VISUAL": "nano",
    
    "PATH": "$PAI_DIR/bin:$HOME/.local/bin:$HOME/.bun/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
  }
}
```

**Key variables:**

| Variable | Purpose | Default |
|----------|---------|---------|
| `PAI_DIR` | PAI installation path | `~/.claude` |
| `MEMORY_DIR` | Persistent memory location | `~/MEMORY` |
| `OLLAMA_HOST` | Ollama server URL | `http://localhost:11434` |
| `BASH_DEFAULT_TIMEOUT_MS` | Command timeout | `600000` (10 min) |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | Max response size | `128000` |

### Permissions

Control what tools can access:

```json
{
  "permissions": {
    "allow": [
      "Read($PAI_DIR/**)",
      "Write($MEMORY_DIR/**)",
      "Bash(git *)",
      "Bash(bun *)"
    ],
    "deny": [
      "Write(/etc/**)",
      "Write(/var/**)",
      "Bash(rm -rf /)",
      "Bash(curl *)"
    ]
  }
}
```

**Syntax:**

- `Read(path)` - File reading
- `Write(path)` - File writing
- `Bash(command)` - Shell commands
- `*` - Wildcard (any)
- `**` - Recursive wildcard

**Variables in paths:**

- `$PAI_DIR` - PAI directory
- `$MEMORY_DIR` - Memory directory
- `$HOME` - User home

**Examples:**

```json
// Allow reading PAI, writing memory
"allow": [
  "Read($PAI_DIR/**)",
  "Write($MEMORY_DIR/**)"
]

// Deny system file writes
"deny": [
  "Write(/etc/**)",
  "Write(/sys/**)",
  "Write(/proc/**)"
]

// Allow safe commands only
"allow": [
  "Bash(git status)",
  "Bash(git diff)",
  "Bash(ls *)"
],
"deny": [
  "Bash(rm *)",
  "Bash(sudo *)"
]
```

### Hooks

Lifecycle event handlers:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "command": "bun $PAI_DIR/hooks/SessionInit.hook.ts",
        "type": "command"
      }
    ],
    
    "UserPromptSubmit": [
      {
        "command": "bun $PAI_DIR/hooks/ModeClassifier.hook.ts",
        "type": "command"
      }
    ],
    
    "PreToolUse": [
      {
        "command": "bun $PAI_DIR/hooks/SecurityValidator.hook.ts",
        "type": "command"
      }
    ],
    
    "Stop": [
      {
        "command": "bun $PAI_DIR/hooks/VoiceAnnounce.hook.ts",
        "type": "command",
        "async": true
      }
    ]
  }
}
```

**Hook types:**

| Hook Point | When it runs | Use for |
|------------|--------------|---------|
| `SessionStart` | Session begins | Load context, check handoffs |
| `UserPromptSubmit` | Before processing request | Mode classification, routing |
| `PreToolUse` | Before tool executes | Security checks, validation |
| `PostToolUse` | After tool completes | Logging, notifications |
| `Stop` | Before response ends | Voice, status updates |
| `SessionEnd` | Session closes | Memory promotion, cleanup |

**Hook options:**

- `async: true` - Don't block on completion
- `timeout: 30000` - Max runtime (ms)
- `if: { ... }` - Conditional execution

### Counts

Auto-updated statistics (read-only):

```json
{
  "counts": {
    "skills": 120,
    "workflows": 206,
    "hooks": 111,
    "sessions": 2415,
    "research": 362,
    "updatedAt": "2026-06-24T21:26:08.609Z"
  }
}
```

Updated by `GetCounts.ts` tool.

### Effort Level

Default task complexity assumption:

```json
{
  "effortLevel": "high"
}
```

**Options:**
- `low` - Quick, simple tasks
- `medium` - Typical work
- `high` - Thorough, detailed (default)

Affects:
- Algorithm phase depth
- VERIFY rigor
- LEARN detail

## User Context

Personal data in `PAI/USER/`:

### Directory Structure

```
PAI/USER/
├── ABOUTME.md                 # Your bio, background
├── DAIDENTITY.md              # AI assistant persona
├── AISTEERINGRULES.md         # Your AI behavior overrides
├── WRITINGSTYLE.md            # Writing preferences
├── PROJECTS/
│   └── PROJECTS.md            # Active projects
├── TELOS/
│   ├── MISSIONS.md            # Life missions
│   ├── GOALS.md               # Current goals
│   ├── BOOKS.md               # Reading list
│   └── WISDOM.md              # Personal wisdom
└── SKILLCUSTOMIZATIONS/       # Skill overrides
```

### ABOUTME.md

Your identity and context:

```markdown
# About Me

## Core Identity
- Name: Your Name
- Location: Your City
- Role: Your profession

## Background
Brief bio, what you do, your interests.

## Current Focus
What you're working on right now.

## Preferences
- Work style: ...
- Communication: ...
- Values: ...
```

### DAIDENTITY.md

AI assistant persona:

```markdown
# Digital Assistant Identity

## Name
Nova

## Persona
Professional, warm, first-person, evidence-first.

## Voice
- Tone: Direct, terse
- Style: Show work via tool output
- Address: Use principal's name
```

### AISTEERINGRULES.md

Your behavioral overrides:

```markdown
# AI Steering Rules - Personal

## Claude Token Conservation
**Statement:** Route to cheapest capable resource first.
**Bad:** Spawning Claude agent for classification.
**Correct:** Route to Ollama, zero token cost.

## Your Custom Rules
Add rules here in Statement/Bad/Correct format.
```

## API Keys and Secrets

### secrets.env Format

```bash
# ~/.config/PAI/secrets.env

# AI Models
ANTHROPIC_API_KEY=sk-ant-xxxxx
OPENAI_API_KEY=sk-xxxxx
GEMINI_API_KEY=xxxxx

# Voice
ELEVENLABS_API_KEY=xxxxx

# Notifications
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/xxxxx
NTFY_TOPIC=your-topic-name

# Services
QDRANT_API_KEY=xxxxx
QDRANT_URL=https://xxxxx.qdrant.tech

# Web Scraping
FIRECRAWL_API_KEY=xxxxx
BROWSERBASE_API_KEY=xxxxx

# Optional: Database
POSTGRES_URL=postgresql://user:pass@host/db
```

### Security

```bash
# Restrict permissions
chmod 600 ~/.config/PAI/secrets.env

# Verify
ls -la ~/.config/PAI/secrets.env
# Should show: -rw------- (600)
```

### Loading Secrets

Secrets loaded automatically by:
- PAI services (Voice Server, OmniPulse)
- Tools that need external APIs
- Never exposed in logs or responses

## Model Configuration

### Inference Matrix

Location: `PAI/Tools/inference-matrix.json`

Maps task types to models:

```json
{
  "fast_classification": {
    "primary": "ollama/llama3.2:3b",
    "fallback": "claude/haiku",
    "timeout": 5000,
    "context_window": 8000
  },
  
  "structured_json": {
    "primary": "ollama/qwen2.5:7b",
    "fallback": "claude/sonnet",
    "schema_support": true
  },
  
  "multi_step_reasoning": {
    "primary": "ollama/deepseek-r1:7b",
    "fallback": "claude/sonnet",
    "timeout": 60000
  },
  
  "code_generation": {
    "primary": "claude/sonnet",
    "fallback": "claude/opus",
    "context_window": 200000
  }
}
```

### Model Routing

Configured in `Inference.ts`:

```bash
# Use the routing
bun PAI/Tools/Inference.ts --task fast_classification "Urgent: yes/no"
bun PAI/Tools/Inference.ts --task structured_json "Extract entities" --schema schema.json
```

### Ollama Models

Install recommended models:

```bash
# Fast (classification, scoring)
ollama pull llama3.2:3b

# Medium (summarization, analysis)
ollama pull gemma2:9b

# Large (extraction, structured output)
ollama pull qwen2.5:7b

# Reasoning
ollama pull deepseek-r1:7b
```

### Model Preferences

In `settings.json`:

```json
{
  "env": {
    "ANTHROPIC_SMALL_FAST_MODEL": "claude-haiku-4-5",
    "ANTHROPIC_CUSTOM_MODEL_OPTION": "http://192.168.50.20:11434/v1,ollama"
  }
}
```

## Skills

### Skill Discovery

Skills auto-indexed from `~/.claude/skills/`:

```bash
# Regenerate index
bun PAI/Tools/GenerateSkillIndex.ts

# Check loaded skills
grep -c "SKILL.md" ~/.claude/skills/*/SKILL.md
```

### Skill Customization

Override skill behavior in `PAI/USER/SKILLCUSTOMIZATIONS/`:

```markdown
# SkillName/OVERRIDE.md

## Custom Workflows

### MyWorkflow
1. Custom step
2. Different approach
3. My preferred tool
```

### Creating Custom Skills

```bash
# Use CreateSkill skill
# In Claude Code:
"Create a new skill for analyzing log files"

# Manual
mkdir ~/.claude/skills/MySkill
nano ~/.claude/skills/MySkill/SKILL.md
```

**SKILL.md template:**

```markdown
# Skill Name

## Triggers
USE WHEN user says X, Y, Z

## Workflows

### MainWorkflow
1. Step 1
2. Step 2
3. Step 3

## Tools
- Tool A
- Tool B

## Outputs
Expected results
```

## Hooks

### Custom Hook Creation

```typescript
// ~/.claude/hooks/handlers/MyHook.hook.ts

import { appendFile } from 'fs/promises';

async function myHook(context: any) {
  const timestamp = new Date().toISOString();
  const logEntry = `${timestamp} - ${context.event}\n`;
  
  await appendFile(
    `${process.env.MEMORY_DIR}/hooks.log`,
    logEntry
  );
}

await myHook(JSON.parse(process.argv[2]));
```

### Register Hook

In `settings.json`:

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "command": "bun hooks/handlers/MyHook.hook.ts",
        "type": "command"
      }
    ]
  }
}
```

### Conditional Hooks

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "command": "bun hooks/SecurityCheck.hook.ts",
        "type": "command",
        "if": {
          "tool": "Bash"
        }
      }
    ]
  }
}
```

## Memory

### Memory Directory Structure

```
~/MEMORY/
├── WORK/              # Session artifacts
│   ├── PRD-*.md
│   └── transcripts/
├── LEARNING/          # Reflections
│   ├── failures/
│   └── patterns/
├── STATE/             # System state
│   ├── session-names.jsonl
│   └── caches/
├── RELATIONSHIP/      # Interaction patterns
│   └── daily/
├── RESEARCH/          # Research outputs
│   └── topics/
└── WISDOM/            # Domain knowledge
    └── frames/
```

### Memory Configuration

```json
{
  "env": {
    "MEMORY_DIR": "/home/user/MEMORY",
    "MEMORY_RETENTION_DAYS": "90",
    "MEMORY_AUTO_ARCHIVE": "true"
  }
}
```

### Memory Limits

```bash
# Archive old sessions
bun PAI/Tools/ArchiveWork.ts --older-than 90

# Check memory usage
ncdu ~/MEMORY

# Prune old learning files
bun PAI/Tools/LearningPrune.ts
```

## Services

### Service Configuration

Systemd user services in `~/.config/systemd/user/`:

```ini
# pai-voice-server.service
[Unit]
Description=PAI Voice Server
After=network.target

[Service]
Type=simple
ExecStart=/home/user/.bun/bin/bun /home/user/.claude/PAI/Tools/VoiceServer.ts
Restart=on-failure
RestartSec=5s
EnvironmentFile=/home/user/.config/PAI/secrets.env

[Install]
WantedBy=default.target
```

### Service Management

```bash
# Enable service
systemctl --user enable pai-voice-server

# Start service
systemctl --user start pai-voice-server

# Check status
systemctl --user status pai-voice-server

# View logs
journalctl --user -u pai-voice-server -f
```

## Advanced

### Multi-Engine Configuration

For running PAI across Claude Code, Antigravity, and Ollama:

```json
{
  "daidentity": {
    "voices": {
      "main": { "voiceId": "kokoro:af_sky" },
      "antigravity": { "voiceId": "am_adam" },
      "ollama": { "voiceId": "am_michael" }
    }
  }
}
```

### Template Variables

In `CLAUDE.md.template`:

```markdown
Principal: {PRINCIPAL.NAME}
Assistant: {DA.NAME}
Algorithm: {ALGORITHM.VERSION}
Skills: {SKILLS.COUNT}
```

Build with:

```bash
bun PAI/Tools/BuildCLAUDE.ts
```

### Dynamic Context Loading

```json
{
  "dynamicContext": {
    "relationship": true,
    "learning": true,
    "work": true
  }
}
```

Toggles context injection at SessionStart.

### Performance Tuning

```json
{
  "env": {
    "BASH_DEFAULT_TIMEOUT_MS": "300000",
    "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "64000",
    "SKILL_INDEX_CACHE_TTL": "3600"
  }
}
```

### Backup Configuration

```bash
# In PAI/Tools/NasBackup.sh
BACKUP_DEST="/mnt/nas/pai-backups"
RETENTION_DAYS=30
COMPRESS=true
```

---

## Configuration Best Practices

1. **Start minimal** - Use defaults, add as needed
2. **Version control** - Git track settings.json (exclude secrets)
3. **Document changes** - Comment why you changed defaults
4. **Test incrementally** - One change at a time
5. **Backup first** - Before major config changes

## Troubleshooting Config

```bash
# Validate settings.json
bun PAI/Tools/ValidateConfig.ts

# Check what's loaded
bun PAI/Tools/ShowLoadedContext.ts

# Test hooks
bun PAI/Tools/HookDispatchSmoke.ts

# Verify permissions
bun PAI/Tools/PermissionTest.ts
```

## Next Steps

- **Deploy PAI**: [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md)
- **Understand architecture**: [ARCHITECTURE.md](ARCHITECTURE.md)
- **Quick start**: [QUICKSTART.md](QUICKSTART.md)
- **Troubleshooting**: [TROUBLESHOOTING_METHOD.md](TROUBLESHOOTING_METHOD.md)

Configuration is power. PAI is yours to shape.
