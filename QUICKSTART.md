# Obsidian Zettelkasten + PAI Bridge — Quickstart

## What You Have Now

A complete Zettelkasten second brain system integrated with PAI:

### Plugins ✅
- **Templates** — Note scaffolding
- **Smart Connections** — AI semantic linking  
- **PAI Bridge** — Auto-sync PAI research/knowledge
- **Graph View** — Visual knowledge map
- **Backlinks** — Bidirectional connections
- **Dataview** — Database queries

### Templates ✅
- `Zettelkasten-Note.md` — Atomic permanent notes
- `Source-Note.md` — Capture from research
- `Daily-Note.md` — Daily reflection

### Folders ✅
- `Zettelkasten/` — Your atomic notes
- `Sources/` — PAI research imports
- `Templates/` — Note templates
- `PAI-Docs/`, `PAI-Knowledge/`, etc. — Existing content

## 30-Second Workflow

### Daily Routine
```
1. Open Obsidian → Click sync icon
2. New Source Notes appear in Sources/
3. Read, extract key ideas
4. Create Zettelkasten notes from selections
5. Link related concepts with [[wikilinks]]
```

### First Time Setup (5 minutes)

1. **Open Obsidian**
   ```bash
   ssh -X duane@pai-primary
   obsidian ~/Projects/pai-quartz/pai-vault-clean/
   ```

2. **Verify plugins loaded**
   - Settings → Community plugins
   - Should see: Dataview, Obsidian Git, Smart Connections, PAI Bridge

3. **First sync**
   - Click sync icon in left ribbon
   - Wait for "Imported N research sessions"

4. **Explore**
   - Check `Sources/` folder
   - Open Graph View (`Cmd/Ctrl+G`)
   - See imported PAI notes

## Key Commands

### Sync & Import
- **Sync PAI Research** — `Cmd/Ctrl+P` → "Sync PAI"
- **Import Knowledge** — `Cmd/Ctrl+P` → "Import PAI"
- **Manual Sync** — Click sync icon in ribbon

### Note Creation  
- **New Zettelkasten note** — Select text → `Cmd/Ctrl+P` → "Create Zettelkasten"
- **Use template** — `Cmd/Ctrl+P` → "Templates: Insert"
- **Daily note** — `Cmd/Ctrl+P` → "Open today's daily note"

### Navigation
- **Graph View** — `Cmd/Ctrl+G`
- **Quick switcher** — `Cmd/Ctrl+O`
- **Command palette** — `Cmd/Ctrl+P`
- **Search** — `Cmd/Ctrl+Shift+F`

### Linking
- **Create link** — Type `[[` → autocomplete
- **View backlinks** — Right sidebar "Backlinks" panel
- **Smart connections** — Right sidebar "Smart Connections" panel

## The Zettelkasten Method (from video)

### Three Rules
1. **Protect thinking** — Capture everything systematically
2. **Expand thinking** — Connect ideas via links
3. **Output-based** — Tags for creating, not filing

### Four-Step Workflow

#### 1. CAPTURE (Input)
Use **Source Notes** for external content:
- PAI research sessions (auto-imported)
- Books, articles, videos (manual)
- Conversations, podcasts

#### 2. PROCESS (Permanent Notes)
Extract **one atomic idea** per note:
- Select key insight from source
- Create Zettelkasten note
- Write in your own words
- Ask: "What does this connect to?"

#### 3. CONNECT (Links)
Build a web, not a hierarchy:
- Use `[[wikilinks]]` liberally
- Smart Connections suggests relationships
- Create Maps of Content (MOCs) for topics
- Graph View shows clusters

#### 4. CREATE (Output)
Use output-based tags:
- `#thinking` — Notes for problem-solving
- `#writing` — Ready to become articles
- `#research` — Needs more investigation  
- `#creating` — For building projects

## PAI Integration

### What Gets Imported

**Research Sessions** (`Sources/`)
- Current month from `~/.claude/PAI/MEMORY/RESEARCH/`
- 188 files available for July 2026
- Auto-tagged: `#pai-import`, `#research`
- Metadata: executor, timestamp, original file

**Knowledge Archive** (`Zettelkasten/`)
- All entries from `~/.claude/PAI/KNOWLEDGE/`
- Categories: Research, People, Companies, Ideas
- Auto-tagged: `#pai-knowledge`, `#category`
- Full content preserved

### Daily Workflow with PAI

```
Morning:
  Run "Sync PAI Research"
  
Processing:
  Review new Source Notes in Sources/
  Extract insights → Zettelkasten notes
  Link to existing concepts
  
Weekly:
  Import PAI Knowledge (slower)
  Review new knowledge entries
  Connect to Zettelkasten
  
Continuous:
  Use Smart Connections sidebar
  Explore Graph View
  Build MOCs
```

## Example: Processing a PAI Research Session

### 1. Source Note Appears
After sync, you see:
```
Sources/2026-07-02-Nova completed Transfer fix...
```

### 2. Read and Extract
Open the note, find a key insight:
> "Transfer linking with balance-unchanged guardrail ensures no dollar moves during recategorization"

### 3. Create Zettelkasten Note
- Select the text
- `Cmd/Ctrl+P` → "Create Zettelkasten note from selection"
- New note opens with template

### 4. Add Your Thinking
```markdown
# Balance Integrity Patterns

## Source
[[2026-07-02-Nova completed Transfer fix]]

## Main Idea
Financial transaction systems should verify balance integrity 
before/after any categorization operation — a guardrail pattern.

## My Thoughts
This is like database ACID properties but for account balances.
Could apply this to budget tracking, not just transfers.
Links to double-entry bookkeeping principle.

## Related Notes
[[Double-Entry Bookkeeping]]
[[ACID Properties]]
[[Budget Categories]]

## References
PAI Research Session 2026-07-02
```

### 5. Link and Discover
- Type `[[` to create links
- Smart Connections suggests related notes
- Graph View shows connections forming

## Power Features

### Dataview Queries
Create dynamic lists:

````markdown
## Recent PAI Imports
```dataview
LIST
FROM #pai-import
SORT created DESC
LIMIT 10
```

## Notes Tagged for Writing
```dataview
LIST
FROM #writing
```
````

### Graph View Filters
- Filter by tag: `#pai-import`
- Hide tag: `-#daily`
- Color groups by tag
- Zoom to see clusters

### Smart Connections
- Sidebar shows semantically related notes
- Uses local AI (privacy-first)
- Suggests connections as you write
- No API keys needed

## Tips

### Starting Out
- Don't obsess over structure — capture first
- One idea per Zettelkasten note
- Links > folders
- Output tags > topic tags

### Daily Habits
- Morning: Sync PAI research
- Reading: Capture to Source Notes
- Thinking: Extract to Zettelkasten
- Evening: Review Graph View

### When Stuck
- Browse Graph View for inspiration
- Check Smart Connections sidebar
- Search for partial keywords
- Review recent Source Notes

## Resources

- **Video:** https://www.youtube.com/watch?v=Aded2v7_vag
- **Testing:** `TEST-PAI-BRIDGE.md`
- **Usage:** `README-ZETTELKASTEN.md`
- **Plugin docs:** `~/Projects/pai-bridge-plugin/README.md`

## Next Steps

1. **Test** — Follow `TEST-PAI-BRIDGE.md` checklist
2. **Sync** — Import your PAI research
3. **Process** — Extract 3-5 Zettelkasten notes
4. **Link** — Connect ideas with `[[wikilinks]]`
5. **Explore** — Open Graph View, see the web form

---

**Setup Date:** 2026-07-15  
**System:** Obsidian + Zettelkasten + PAI Bridge  
**Status:** Ready to use ✅
