# Syncing Obsidian Vault to Laptop

## Overview

Your vault is already a git repository:
- **GitHub Remote:** `https://github.com/dcloakdc-source/pai-knowledge-vault.git`
- **pai-primary Location:** `~/Projects/pai-quartz/pai-vault-clean/`
- **Laptop Connection:** SSH/SFTP access to pai-primary

## Best Sync Method: Git + Obsidian Git Plugin

The vault already has **Obsidian Git plugin** installed. This gives you automatic bidirectional sync.

### Architecture

```
pai-primary                          GitHub                          Laptop
    ↓                                   ↓                              ↓
pai-vault-clean/             github.com/dcloakdc-source/      C:\ObsidianVaults\pai-vault-clean\
    ↓                        pai-knowledge-vault                      ↓
Obsidian Git plugin    ←───→        origin        ←───→      Obsidian Git plugin
Auto-commit/push                                             Auto-commit/pull
```

## Setup on Laptop (One-Time)

### Step 1: Install Obsidian on Laptop
Download from https://obsidian.md

### Step 2: Clone the Vault
```bash
# Windows (PowerShell or Git Bash)
cd C:\ObsidianVaults\
git clone https://github.com/dcloakdc-source/pai-knowledge-vault.git pai-vault-clean
cd pai-vault-clean
```

### Step 3: Open in Obsidian
1. Launch Obsidian
2. **Open folder as vault**
3. Select `C:\ObsidianVaults\pai-vault-clean\`
4. Trust the vault (it will load plugins)

### Step 4: Verify Plugins Loaded
**Settings → Community plugins**

Should auto-load:
- ✅ Dataview
- ✅ Obsidian Git
- ✅ Smart Connections
- ✅ PAI Bridge

**If plugins don't auto-enable:**
- Settings → Community plugins → Turn off "Restricted mode"
- Reload Obsidian (Ctrl+R)
- Enable each plugin manually

### Step 5: Configure PAI Bridge (Laptop-Specific)

**Problem:** PAI paths on laptop are different from pai-primary

**Solution:** Update PAI Bridge settings to point to SSH-mounted or synced PAI directories

**Option A: SSH Mount (Windows)**
```powershell
# Install SSHFS-Win or WinFsp + SSHFS
# Mount pai-primary PAI directory
sshfs duane@pai-primary:/home/duane/.claude/PAI P:\PAI
```

Then in Obsidian:
- Settings → PAI Bridge
- PAI Research Path: `P:\PAI\MEMORY\RESEARCH`
- PAI Knowledge Path: `P:\PAI\KNOWLEDGE`

**Option B: Disable PAI Sync on Laptop**
If you only want to sync from pai-primary:
- Settings → PAI Bridge
- Leave paths as-is (they'll fail on laptop)
- Only use sync feature on pai-primary
- Laptop gets imported notes via git

**Option C: WSL2 Access (if using WSL)**
```bash
# In WSL
cd /mnt/c/ObsidianVaults/pai-vault-clean
```
Access PAI via SSH mount in WSL, then Obsidian reads from `/mnt/c/`

## Daily Workflow

### On pai-primary (Morning)
1. Open Obsidian
2. Click PAI Bridge sync icon
3. 188 research notes imported to `Sources/`
4. Obsidian Git auto-commits and pushes to GitHub

### On Laptop (Anytime)
1. Open Obsidian
2. Obsidian Git auto-pulls from GitHub
3. New Source Notes appear
4. Process notes, create Zettelkasten notes
5. Obsidian Git auto-commits and pushes your work

### Back on pai-primary (Evening)
1. Open Obsidian
2. Obsidian Git auto-pulls your laptop work
3. See your new Zettelkasten notes
4. Continue working

## Obsidian Git Plugin Settings

**Settings → Obsidian Git** (configure on both machines)

Recommended settings:
```
✅ Auto pull on startup (default: every 10 minutes)
✅ Auto commit (default: every 10 minutes)
✅ Auto push (default: after commit)
   Commit message: "vault backup: {{date}}"
   
⚠️  Pull updates on startup: ON
⚠️  Disable push: OFF
⚠️  Disable notifications: Optional (can be noisy)
```

## Conflict Resolution

### If Both Machines Edit Same File
Git will create a conflict. Obsidian Git shows notification.

**To Resolve:**
1. `Cmd/Ctrl+P` → "Obsidian Git: Open source control view"
2. See conflicted files
3. Click file → choose version (yours/theirs) or manually merge
4. Commit resolution

**Prevention:**
- Work on different notes on different machines
- Always pull before starting work
- Use PAI sync only on pai-primary

## PAI Bridge Plugin Sync

### Option 1: Sync Only on pai-primary (Recommended)
**Why:** PAI directories live on pai-primary

**Setup:**
- pai-primary: Run PAI sync, creates Source Notes
- Git pushes to GitHub
- Laptop pulls from GitHub
- Laptop gets imported notes without needing PAI access

**Workflow:**
```
pai-primary:
  1. PAI Bridge syncs → Source Notes created
  2. Obsidian Git auto-commits → pushed to GitHub

Laptop:
  1. Obsidian Git auto-pulls → Source Notes appear
  2. Process notes, create Zettelkasten notes
  3. Obsidian Git auto-commits → pushed to GitHub

pai-primary:
  1. Obsidian Git auto-pulls → Zettelkasten notes appear
```

### Option 2: SSH Mount PAI Directories (Advanced)
**Why:** Want to sync PAI from laptop too

**Setup on Windows:**
```powershell
# Install SSHFS-Win: https://github.com/winfsp/sshfs-win
# Or: scoop install sshfs-np

# Mount
net use P: \\sshfs\duane@pai-primary\.claude\PAI
```

**Then configure PAI Bridge:**
- Settings → PAI Bridge
- PAI Research Path: `P:\MEMORY\RESEARCH`
- PAI Knowledge Path: `P:\KNOWLEDGE`

**Caveat:** SSH mount can be slow for 188-file sync

### Option 3: Disable on Laptop (Simplest)
**Settings → Community plugins → PAI Bridge → Disable**

Only use sync feature on pai-primary. Laptop is read-only for PAI imports.

## Plugin Installation on Laptop

The PAI Bridge plugin files will sync via git:
```
.obsidian/plugins/pai-bridge/
├── main.js
├── manifest.json
└── styles.css
```

**They should auto-load on laptop.**

If not:
- Settings → Community plugins → Reload
- Or manually enable "PAI Bridge"

## Folder Behavior

### What Syncs via Git ✅
- `.obsidian/` (plugins, settings)
- `Templates/`
- `Sources/` (PAI imports from pai-primary)
- `Zettelkasten/` (your notes from either machine)
- `PAI-Docs/`, `PAI-Knowledge/`, etc.
- All markdown files

### What Doesn't Sync ❌
- Plugin cache files (ignored via .gitignore)
- Smart Connections embeddings (regenerates on each machine)
- Workspace state (per-machine)

## Testing Laptop Sync

### Step-by-Step Test

**On pai-primary:**
1. Open vault, create test note: `Test-From-Server.md`
2. Write: "This note was created on pai-primary"
3. Wait 10 minutes (or `Cmd/Ctrl+P` → "Obsidian Git: Commit and push")

**On laptop:**
1. Open vault
2. Wait for auto-pull (or `Cmd/Ctrl+P` → "Obsidian Git: Pull")
3. Verify `Test-From-Server.md` appears

**On laptop:**
1. Create note: `Test-From-Laptop.md`
2. Write: "This note was created on laptop"
3. Wait for auto-commit/push

**On pai-primary:**
1. Wait for auto-pull
2. Verify `Test-From-Laptop.md` appears

**Success:** Bidirectional sync working ✅

## Troubleshooting

### "PAI Bridge: Path not found" on Laptop
**Normal.** PAI directories don't exist on laptop.
- Disable plugin on laptop, or
- Configure SSH mount, or
- Ignore the error (PAI sync only works on pai-primary)

### Obsidian Git Not Auto-Committing
- Settings → Obsidian Git → Check "Auto commit" is enabled
- Check commit interval (default: 10 minutes)
- Manual test: `Cmd/Ctrl+P` → "Obsidian Git: Commit all changes"

### Plugins Don't Load on Laptop
- Settings → Community plugins → "Restricted mode" OFF
- Reload Obsidian (Ctrl+R)
- Check `.obsidian/community-plugins.json` exists

### Merge Conflicts
- Use Obsidian Git's built-in conflict resolver
- Or: resolve in external git tool
- Commit resolution

### Slow Git Operations
- Large vault (179+ files) can slow commits
- Consider selective sync (git sparse-checkout)
- Or: sync only active folders

## Alternative: Manual Sync via rsync/SFTP

If git sync has issues, use file sync:

**From pai-primary to laptop:**
```bash
# On laptop (WSL or Git Bash)
rsync -avz --delete duane@pai-primary:~/Projects/pai-quartz/pai-vault-clean/ \
  /c/ObsidianVaults/pai-vault-clean/
```

**From laptop to pai-primary:**
```bash
rsync -avz --delete /c/ObsidianVaults/pai-vault-clean/ \
  duane@pai-primary:~/Projects/pai-quartz/pai-vault-clean/
```

**Caveat:** Manual, error-prone, no conflict resolution

## Recommended Setup Summary

### On pai-primary
- ✅ Use PAI Bridge sync (you have PAI directories)
- ✅ Enable Obsidian Git auto-commit/push
- ✅ Process PAI imports, create notes

### On laptop
- ✅ Clone vault from GitHub
- ✅ Enable Obsidian Git auto-pull
- ❌ Disable PAI Bridge (or leave enabled, ignore errors)
- ✅ Process notes, create Zettelkasten notes
- ✅ Obsidian Git auto-commits your work

### Result
- pai-primary: Imports PAI content, creates Source Notes
- GitHub: Central sync hub
- Laptop: Receives Source Notes, creates Zettelkasten notes
- Both machines: Bidirectional sync via Git

---

**Setup Time:** 10 minutes on laptop  
**Sync Method:** Git (automatic via Obsidian Git plugin)  
**PAI Access:** pai-primary only (recommended)  
**Conflict Resolution:** Obsidian Git built-in
