# PAI Knowledge Vault - Deployment Info

## Live URLs

**Primary:** https://quartz.cloudenz.org  
**Cloudflare:** https://pai-knowledge.pages.dev (automatic subdomain)

## Deployment Details

- **Platform:** Cloudflare Pages
- **Project Name:** pai-knowledge
- **Account ID:** 3a7266f44a39cd3458824597a7837830
- **Zone:** cloudenz.org (1c5c89f35cf18dfaf3f7454edcdb1256)
- **DNS:** CNAME quartz.cloudenz.org → pai-knowledge.pages.dev (proxied)
- **SSL:** Active (Cloudflare Universal SSL)
- **Build Tool:** Quartz (static site generator)

## Repository

- **GitHub:** https://github.com/dcloakdc-source/pai-knowledge-vault
- **Branch:** main
- **Local Vault:** ~/Projects/pai-quartz/pai-vault-clean/
- **Quartz Build:** ~/Projects/pai-quartz/

## Auto-Deployment

**Vault → Git:**
- Systemd timer: `vault-git-sync.timer` (every 10 minutes)
- Git auto-commit via Obsidian Git plugin (10min interval)
- Location: `~/.config/systemd/user/vault-git-sync.service`

**Git → Build → Deploy:**
- Systemd service: `quartz-auto-rebuild.service`
- Watches vault for changes (inotifywait)
- Debounce: 60 seconds
- Auto-builds: `npx quartz build`
- Auto-deploys: `wrangler pages deploy`
- Location: `~/.config/systemd/user/quartz-auto-rebuild.service`

**Status Checks:**
```bash
# Check auto-rebuild status
systemctl --user status quartz-auto-rebuild.service

# Check git sync status
systemctl --user status vault-git-sync.timer

# View logs
journalctl --user -u quartz-auto-rebuild.service -f
journalctl --user -u vault-git-sync.service -f
```

## Manual Operations

**Rebuild and Deploy:**
```bash
cd ~/Projects/pai-quartz
npx quartz build
source ~/.config/PAI/secrets.env
wrangler pages deploy public --project-name=pai-knowledge
```

**Git Push:**
```bash
cd ~/Projects/pai-quartz/pai-vault-clean
git add .
git commit -m "Update vault content"
git push origin main
```

**Test Build Locally:**
```bash
cd ~/Projects/pai-quartz
npx quartz build --serve
# Opens at http://localhost:8080
```

## Vault Structure

```
pai-vault-clean/
├── PAI-Docs/          # PAI documentation (100+ files)
├── PAI-Knowledge/     # Knowledge base entries
├── PAI-Projects/      # Project documentation
├── PAI-User/          # User context files
├── PAI-NAS/           # NAS documentation
├── LEARNING/          # Learning resources
├── Research/          # Research notes
├── SECURITY/          # Security documentation
└── OBSERVABILITY/     # Monitoring docs
```

**Excluded:**
- MEMORY/WORK/ (ISA YAML frontmatter issues)
- .obsidian/ (local workspace state)

## Cloudflare Pages Settings

**Build Command:** `npx quartz build`  
**Build Output:** `public/`  
**Root Directory:** `/`  
**Node Version:** 18 (auto-detected)

## DNS Configuration

```
Type: CNAME
Name: quartz
Content: pai-knowledge.pages.dev
TTL: Auto
Proxy: Yes (Orange cloud)
```

## Deployment History

- **2026-07-09:** Initial deployment to pai-knowledge.pages.dev
- **2026-07-10:** Custom domain added: quartz.cloudenz.org
- **2026-07-10:** Auto-rebuild service configured
- **2026-07-10:** Git auto-sync configured

## Future Windows Recovery Vault

A separate vault is ready at `~/Projects/windows-recovery-vault` for Windows drive recovery content.

**Recovery Scripts:**
- `/share/PAI/Scripts/windows-recovery-scan.sh` - Scan Windows drive
- `/share/PAI/Scripts/windows-to-vault-sync.sh` - Sync to vault
- `/share/PAI/Scripts/WINDOWS-RECOVERY-INSTRUCTIONS.md` - Full guide

**Laptop Access Scripts:**
- `/share/PAI/Scripts/mount-windows-recovery-vault.sh` - SSHFS mount
- `/share/PAI/Scripts/clone-windows-recovery-vault.sh` - Clone to laptop

---

**Maintained by:** PAI Nova  
**Last Updated:** 2026-07-10
