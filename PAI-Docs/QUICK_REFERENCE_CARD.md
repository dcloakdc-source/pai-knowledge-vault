# PAI Quick Reference Card

**Last Updated:** 2026-06-30  
**Version:** 5.1.0  
**Print this page for desk reference**

---

## 🚨 Emergency Commands

```bash
# System health
systemctl --user status pai-primary
systemctl --user status ollama

# Restart services
systemctl --user restart pai-primary
systemctl --user restart ollama

# Check logs
journalctl --user -u pai-primary -f
journalctl --user -u ollama -f

# Troubleshooting
bun ~/.claude/PAI/Tools/HealthCheck.ts
```

---

## 📊 Agent Loop Monitoring

```bash
# Start monitoring
bun ~/.claude/PAI/Tools/AgentLoopMonitor.ts start \
  --orchestrator "main" --pattern "research"

# Log iteration
bun ~/.claude/PAI/Tools/AgentLoopMonitor.ts iteration <loop-id> \
  --status success --tokens 1200

# End loop
bun ~/.claude/PAI/Tools/AgentLoopMonitor.ts end <loop-id>

# Dashboard
bun ~/.claude/PAI/Tools/AgentLoopDashboard/server.ts
# → http://localhost:3000
```

---

## 💰 Cost Tracking

```bash
# Record cost
bun ~/.claude/PAI/Tools/Cost.ts record \
  <session> <model> <in-tokens> <out-tokens>

# Daily report
bun ~/.claude/PAI/Tools/Cost.ts report --period daily

# Weekly report
bun ~/.claude/PAI/Tools/Cost.ts report --period weekly

# Session cost
bun ~/.claude/PAI/Tools/Cost.ts report --session <id>
```

---

## 📁 Project Management

```bash
# Switch project
bun ~/.claude/PAI/Tools/ProjectManager.ts switch <project>

# Park project (save for later)
bun ~/.claude/PAI/Tools/ProjectManager.ts park <project>

# Resume project (restore context)
bun ~/.claude/PAI/Tools/ProjectManager.ts resume <project>

# List all projects
bun ~/.claude/PAI/Tools/ProjectManager.ts list

# Show active
bun ~/.claude/PAI/Tools/ProjectManager.ts active
```

---

## 🎯 Focus Management (ADHD)

```bash
# Start focus session
bun ~/.claude/PAI/Tools/FocusManager.ts start <task> \
  --energy high|medium|low

# Capture distraction (without breaking flow)
bun ~/.claude/PAI/Tools/FocusManager.ts capture "<thought>"

# Complete session
bun ~/.claude/PAI/Tools/FocusManager.ts complete

# View stats
bun ~/.claude/PAI/Tools/FocusManager.ts stats

# Resume interrupted session
bun ~/.claude/PAI/Tools/FocusManager.ts resume
```

---

## ✅ Approval Gates

```bash
# One-time TOTP setup
bun ~/.claude/PAI/Tools/ApprovalGate.ts --setup
# → Scan QR code with Google Authenticator

# Request approval (dual-factor)
bun ~/.claude/PAI/Tools/ApprovalGate.ts --request <slug> <action>

# Verify TOTP only
bun ~/.claude/PAI/Tools/ApprovalGate.ts --verify-totp <code>

# Check audit log integrity
bun ~/.claude/PAI/Tools/ApprovalGate.ts --verify-log
```

---

## 🔗 Cross-Domain Linking

```bash
# Detect patterns across files
bun ~/.claude/PAI/Tools/CrossDomainLinker.ts detect <file1> <file2> [...]

# Synthesize domains
bun ~/.claude/PAI/Tools/CrossDomainLinker.ts synthesize <domain1> <domain2>

# Run demo
bun ~/.claude/PAI/Tools/CrossDomainLinker.demo.ts
```

---

## 🧠 Memory & Knowledge

```bash
# Query memory
bun ~/.claude/PAI/Tools/MemoryQuery.ts "<query>"

# Harvest session
bun ~/.claude/PAI/Tools/SessionHarvester.ts

# Knowledge graph
bun ~/.claude/PAI/Tools/WikilinkGraph.ts generate

# Bulk index
bun ~/.claude/PAI/Tools/BulkIndexPAIUser.ts
```

---

## 🎬 Media & Content

```bash
# Monitor feeds
bun ~/.claude/PAI/Tools/FeedMonitor.ts check

# Process inbox
bun ~/.claude/PAI/Tools/InboxAutoProcessor.ts

# Media workflow
bun ~/.claude/PAI/Tools/MediaWorkflow.ts process <file>
```

---

## 🔍 Troubleshooting Checklist

**Service not running?**
```bash
systemctl --user status <service>
systemctl --user start <service>
journalctl --user -u <service> -n 50
```

**High cost?**
```bash
bun ~/.claude/PAI/Tools/Cost.ts report --period daily
# Check for runaway loops
```

**Lost context?**
```bash
bun ~/.claude/PAI/Tools/ProjectManager.ts resume <project>
cat ~/.claude/PAI/MEMORY/STATE/projects-registry.json
```

**Focus session interrupted?**
```bash
bun ~/.claude/PAI/Tools/FocusManager.ts resume
```

**Agent loop stuck?**
```bash
# Check dashboard
bun ~/.claude/PAI/Tools/AgentLoopDashboard/server.ts
# → http://localhost:3000
```

---

## 📂 Key File Locations

```bash
# Configuration
~/.claude/settings.json                     # Main config
~/.claude/PAI/config/                       # PAI config

# State
~/MEMORY/STATE.claude/                      # State files
~/MEMORY/db/                                # Databases

# Logs
~/MEMORY/STATE.claude/agent-loops.jsonl     # Loop logs
~/MEMORY/STATE.claude/destructive-promotions.jsonl  # Approvals

# Projects
~/.claude/PAI/MEMORY/STATE/projects-registry.json   # Registry
<project-root>/.project/                    # Per-project state

# Documentation
~/.claude/PAI/DOCUMENTATION/INDEX.md        # Master index
~/.claude/PAI/DOCUMENTATION/                # All docs
```

---

## 🎛️ Service Status

```bash
# All services
systemctl --user list-units 'pai-*' 'ollama*'

# Individual status
systemctl --user status pai-primary
systemctl --user status ollama
systemctl --user status llamacpp-native

# Enable auto-start
systemctl --user enable pai-primary
systemctl --user enable ollama
```

---

## 📊 Performance Monitoring

```bash
# System health
bun ~/.claude/PAI/Tools/HealthCheck.ts

# GPU status
nvidia-smi

# Ollama models
ollama list

# Memory usage
free -h

# Disk space
df -h ~/MEMORY
```

---

## 🔑 Common Patterns

**Start new project:**
```bash
cd /path/to/project
bun ~/.claude/PAI/Tools/ProjectManager.ts switch "$(basename $PWD)"
```

**Switch between projects:**
```bash
bun ~/.claude/PAI/Tools/ProjectManager.ts list
bun ~/.claude/PAI/Tools/ProjectManager.ts switch <name>
```

**Monitor long-running task:**
```bash
LOOP_ID=$(bun ~/.claude/PAI/Tools/AgentLoopMonitor.ts start \
  --orchestrator "main" --pattern "research")
# ... do work ...
bun ~/.claude/PAI/Tools/AgentLoopMonitor.ts end $LOOP_ID
```

**Focus session workflow:**
```bash
# Morning
bun ~/.claude/PAI/Tools/FocusManager.ts start "Deep work" --energy high

# Distraction appears
bun ~/.claude/PAI/Tools/FocusManager.ts capture "Check email about X"

# After work
bun ~/.claude/PAI/Tools/FocusManager.ts complete
bun ~/.claude/PAI/Tools/FocusManager.ts stats
```

---

## 📞 Quick Help

**Full documentation:**
```bash
less ~/.claude/PAI/DOCUMENTATION/INDEX.md
```

**Tool help:**
```bash
bun ~/.claude/PAI/Tools/<Tool>.ts --help
```

**Skills:**
```bash
ls -1 ~/.claude/skills/*/SKILL.md
```

**Current session:**
```bash
echo $CLAUDE_SESSION_ID
```

---

## 🎯 Philosophy Reminders

**Projects:**
> Unfinished projects are exploration, not failure.  
> Context-switching is a feature, not a bug.

**Focus (ADHD):**
> ADHD isn't a bug — it's a different operating system.  
> Work WITH attention patterns, not against them.

**Costs:**
> Route to cheapest capable resource.  
> Claude tokens are the scarcest resource.

**Approval:**
> Dual-factor for all destructive operations.  
> Audit trails are tamper-evident by design.

---

**Print Date:** $(date +"%Y-%m-%d")  
**Version:** PAI 5.1.0  
**Engine:** PNK (OpenCode)
