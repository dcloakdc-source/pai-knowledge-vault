# Zettelkasten Setup Guide

This vault is now configured for the Zettelkasten method as demonstrated in Vicky Zhao's "Obsidian: The Thinking Person's Set Up" tutorial.

## What's Enabled

### Core Plugins ✅
- **Templates** — For note scaffolding
- **Backlinks** — Bidirectional linking
- **Graph View** — Visual knowledge map
- **Daily Notes** — Daily capture
- **Tag Pane** — Tag-based navigation

### Community Plugins ✅
- **Smart Connections v4.5.3** — AI-powered semantic linking (zero-setup, local embeddings)
- **Dataview** — Database queries over notes
- **Obsidian Git** — Auto version control

## Templates Created

Located in `Templates/`:

1. **Zettelkasten-Note.md** — Atomic permanent notes (one idea per note)
2. **Source-Note.md** — Capture from books, articles, videos, podcasts
3. **Daily-Note.md** — Daily capture and reflection

## Folder Structure

```
pai-vault-clean/
├── Zettelkasten/         ← Your atomic permanent notes go here
├── Templates/            ← Note templates
├── PAI-Docs/             ← Technical documentation
├── PAI-Knowledge/        ← Curated knowledge
├── PAI-Projects/         ← Project tracking
├── PAI-NAS/              ← NAS-synced content
└── index.md              ← Vault home page
```

## The Zettelkasten Workflow

### 1. Capture (Input)
- Use **Source Notes** for books, articles, videos, podcasts
- Use **Daily Notes** for thoughts, observations, conversations
- Don't worry about structure yet — just capture

### 2. Process (Permanent Notes)
- Extract ONE atomic idea from your source
- Create a **Zettelkasten Note** in your own words
- Link to the source note
- Ask: "What does this idea connect to?"

### 3. Connect (Links)
- Use [[wikilinks]] to connect related ideas
- Add to existing Maps of Content (MOCs)
- Let Smart Connections suggest relationships
- Build a web, not a hierarchy

### 4. Create (Output)
- Output-based tags: `#thinking` `#writing` `#research` `#creating`
- Use connected notes to write articles, essays, projects
- The system should help you THINK, not just STORE

## Using Templates

In Obsidian GUI:
1. Create new note
2. `Cmd/Ctrl + P` → "Templates: Insert template"
3. Choose Zettelkasten-Note, Source-Note, or Daily-Note
4. Fill in the scaffolding

## Smart Connections

On first launch in Obsidian:
1. Plugin will auto-index your vault
2. As you write, sidebar shows related notes
3. Uses local AI — no API keys needed
4. Private — never leaves your machine

## Graph View

- `Cmd/Ctrl + G` to open graph
- Click nodes to navigate
- Filter by tags
- Zoom to see clusters of related ideas

## Tag Strategy (Output-Based)

From the video — don't organize by topic, organize by USE:

- `#thinking` — Notes that help you think through problems
- `#writing` — Notes ready to become articles/essays
- `#research` — Notes that need more investigation
- `#creating` — Notes for building projects

Avoid topic tags like `#ai` or `#productivity` — those become filing cabinets you never open.

## Next Steps

1. Open vault in Obsidian GUI (laptop or X11 forwarding)
2. Verify plugins are active
3. Create your first Zettelkasten note from this README
4. Watch the video again (timestamp 12:59 for Zettelkasten method)

## Resources

- Original video: https://www.youtube.com/watch?v=Aded2v7_vag
- Smart Connections: https://smartconnections.app
- Zettelkasten method: https://zettelkasten.de/posts/overview/

---

Setup completed: 2026-07-14  
Configuration: Headless via SSH (JSON editing + GitHub releases)
