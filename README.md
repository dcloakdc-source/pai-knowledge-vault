# PAI Knowledge Vault

Personal AI Infrastructure knowledge base — comprehensive documentation, memory, projects, and research.

## 📚 Contents

- **PAI-Docs/** — System architecture, algorithms, documentation
- **PAI-Knowledge/** — Curated knowledge (Ideas, Research)
- **PAI-Projects/** — Active projects inventory
- **PAI-User/** — Personal context and preferences
- **PAI-NAS/** — NAS deployment documentation

## 🌐 Live Site

Published at: **https://pai-knowledge.pages.dev**

Built with [Quartz](https://quartz.jzhao.xyz) and deployed to Cloudflare Pages.

## 🔄 Sync Workflow

1. Edit notes in Obsidian
2. Obsidian Git auto-commits (10 min interval)
3. Changes push to GitHub
4. pai-primary auto-rebuilds Quartz
5. Auto-deploys to Cloudflare Pages

## 🚀 Setup

### Clone in Obsidian (Recommended)

1. Install **Obsidian Git** plugin
2. Command Palette → "Clone an existing remote repo"
3. Enter: `https://github.com/dcloakdc-source/pai-knowledge-vault.git`
4. Choose local folder
5. Vault opens automatically

### Manual Clone

```bash
git clone https://github.com/dcloakdc-source/pai-knowledge-vault.git
# Open folder in Obsidian: File → Open vault
```

## ⚙️ Configuration

The vault includes pre-configured:
- Core plugins (file explorer, graph, search, backlinks)
- Community plugins (Dataview, Obsidian Git)
- Auto-commit every 10 minutes
- Minimal theme (dark mode)

## 📝 Editing

- **Local edits:** Sync via Obsidian Git (auto or manual)
- **Web preview:** Changes appear on pai-knowledge.pages.dev within ~2 minutes
- **Collaboration:** Fork repo, submit PRs for contributions

---

**Version:** 2026-07-07  
**Obsidian Version:** v1.12.7+  
**License:** Personal knowledge base
