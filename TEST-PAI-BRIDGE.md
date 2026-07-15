# Testing PAI Bridge Plugin v0.2.0

## Current Status ✅

Everything is installed and ready to test:

- ✅ Obsidian installed on pai-primary (`/home/duane/.local/bin/obsidian`)
- ✅ Vault configured at `~/Projects/pai-quartz/pai-vault-clean/`
- ✅ Templates plugin enabled
- ✅ Smart Connections v4.5.3 installed
- ✅ PAI Bridge v0.2.0 installed and enabled
- ✅ Templates created (Zettelkasten-Note, Source-Note, Daily-Note)
- ✅ `Sources/` and `Zettelkasten/` folders created

## How to Open Obsidian

### Option 1: X11 Forwarding (from laptop)
```bash
# From your Windows laptop in WSL or Git Bash
ssh -X duane@pai-primary
obsidian ~/Projects/pai-quartz/pai-vault-clean/
```

### Option 2: Open on laptop directly
If you have the vault synced via git/network:
```bash
# Open Obsidian app
# File → Open vault
# Navigate to pai-vault-clean
```

## Testing Checklist

### 1. Verify Plugins Loaded ✅

**Settings → Community plugins**

Should see:
- ✅ Dataview
- ✅ Obsidian Git  
- ✅ Smart Connections v4.5.3
- ✅ PAI Bridge v0.2.0

**Settings → Core plugins**

Should see:
- ✅ Templates (enabled)
- ✅ Backlinks (enabled)
- ✅ Graph (enabled)

### 2. Test PAI Research Sync

**Step 1: Run Sync**
- Click the **sync icon** in left ribbon (or)
- `Cmd/Ctrl + P` → type "Sync PAI Research" → Enter

**Expected Result:**
- Notice: "Syncing PAI research..."
- Notice: "Imported N research sessions"
- Files appear in `Sources/` folder

**Step 2: Verify Source Note**
- Navigate to `Sources/` folder
- Open any note (e.g., `2026-07-02-Nova completed Transfer fix...`)

**Check:**
- ✅ Frontmatter has `pai-executor: nova`
- ✅ Frontmatter has `pai-original-file: 2026-07-02-044948_AGENT...`
- ✅ Tags include `#pai-import`, `#research`
- ✅ Content has "Source", "Summary", "Original Content" sections

### 3. Test Knowledge Import

**Step 1: Run Import**
- `Cmd/Ctrl + P` → "Import PAI Knowledge" → Enter

**Expected Result:**
- Notice: "Importing PAI knowledge base..."
- Notice: "Imported N knowledge entries"
- Files appear in `Zettelkasten/` folder

**Step 2: Verify Knowledge Note**
- Navigate to `Zettelkasten/` folder
- Open any note starting with `Research-`

**Check:**
- ✅ Frontmatter has `pai-category: Research`
- ✅ Tags include `#pai-knowledge`
- ✅ Content from PAI Knowledge Archive preserved

### 4. Test Zettelkasten Note Creation

**Step 1: Create from Selection**
- Open any note (e.g., `README-ZETTELKASTEN.md`)
- Select a paragraph of text
- `Cmd/Ctrl + P` → "Create Zettelkasten note from selection" → Enter

**Expected Result:**
- New note opens in editor
- Filename: timestamp-based (e.g., `2026-07-15T10-13-00-note.md`)
- Template filled with your selected text
- Note saved in `Zettelkasten/` folder

**Check:**
- ✅ Frontmatter has `created`, `tags`, `aliases`
- ✅ "Main Idea" section has your selected text
- ✅ Other sections empty (for you to fill)

### 5. Test Templates

**Step 1: Create Note from Template**
- Create new note anywhere
- `Cmd/Ctrl + P` → "Templates: Insert template" → Enter
- Choose "Zettelkasten-Note"

**Expected Result:**
- Template content inserted
- `{{title}}`, `{{date}}` variables replaced

**Check:**
- ✅ Three templates available (Zettelkasten, Source, Daily)
- ✅ Variables auto-filled with current date/time

### 6. Test Smart Connections

**Step 1: Wait for Indexing**
- On first launch, Smart Connections indexes your vault
- Watch status bar for "Indexing..." message
- Takes 30-60 seconds for ~200 notes

**Step 2: View Suggestions**
- Open any note
- Look at right sidebar
- "Smart Connections" panel should show related notes

**Check:**
- ✅ Related notes appear in sidebar
- ✅ PAI-imported notes included in suggestions
- ✅ Relevance scores shown

### 7. Test Graph View

**Step 1: Open Graph**
- `Cmd/Ctrl + G` (or)
- Click graph icon in left ribbon

**Expected Result:**
- Graph visualization of all notes
- Nodes = notes, edges = links

**Step 2: Filter by PAI Content**
- In graph panel, click "Filters"
- Under "Tags", add: `#pai-import`

**Check:**
- ✅ Graph shows only PAI-imported notes
- ✅ Connections between PAI research/knowledge visible
- ✅ Can click nodes to open notes

### 8. Test Dataview (Optional)

**Create a test note with this query:**

````markdown
# PAI Research Dashboard

## All PAI Imports
```dataview
LIST
FROM #pai-import
SORT created DESC
LIMIT 10
```

## By Executor
```dataview
TABLE pai-executor, created
FROM #pai-import
WHERE pai-executor
```
````

**Expected Result:**
- Lists of imported notes appear
- Tables show metadata from frontmatter

## Troubleshooting

### Plugin Doesn't Appear
- Check: Settings → Community plugins → "Restricted mode" is OFF
- Reload Obsidian (Cmd/Ctrl+R)
- Check console: `Cmd/Ctrl+Shift+I` → Console tab

### Sync Shows "Path not found"
- Settings → PAI Bridge → Verify paths:
  - Research: `/home/duane/.claude/PAI/MEMORY/RESEARCH`
  - Knowledge: `/home/duane/.claude/PAI/KNOWLEDGE`
- On pai-primary, verify: `ls ~/.claude/PAI/MEMORY/RESEARCH/2026-07/`

### No Notes Created
- Open Console (`Cmd/Ctrl+Shift+I`)
- Run sync again
- Check for error messages
- Verify: `ls ~/Projects/pai-quartz/pai-vault-clean/Sources/`

### Smart Connections Not Working
- Settings → Community plugins → Smart Connections
- Click "Re-index vault"
- Wait for completion

## Expected File Count

After testing all syncs:

```
pai-vault-clean/
├── Sources/              (~188 notes from July research)
├── Zettelkasten/         (~6+ notes from Knowledge)
├── Templates/            (3 templates)
└── [existing folders]
```

## What to Look For

### Success Indicators ✅
- Source Notes in `Sources/` with PAI metadata
- Knowledge notes in `Zettelkasten/` with category tags
- Graph shows interconnected PAI + manual notes
- Smart Connections suggests related content
- Templates work when inserting
- No console errors

### Known Behaviors (Not Bugs)
- Re-running sync skips duplicates (normal)
- Only current month synced (2026-07)
- Large imports take 1-2 minutes (normal)
- Smart Connections re-indexes after import (normal)

## Next Steps After Testing

Once everything works:

1. **Daily workflow:** Run "Sync PAI Research" each morning
2. **Processing:** Extract insights into Zettelkasten notes
3. **Linking:** Connect ideas with [[wikilinks]]
4. **Creating:** Build MOCs and output from connected notes

## Getting Help

If something doesn't work:

1. Check Console for errors
2. Review `USAGE.md` for detailed workflows
3. Check `DEVELOPMENT.md` for known limitations
4. Look at plugin source: `~/Projects/pai-bridge-plugin/src/main.ts`

---

**Testing Date:** 2026-07-15  
**Plugin Version:** PAI Bridge v0.2.0  
**Vault:** pai-vault-clean  
**Expected Import Count:** 188+ research, 6+ knowledge
