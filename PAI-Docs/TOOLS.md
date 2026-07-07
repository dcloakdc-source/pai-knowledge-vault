# PAI Tools - CLI Utilities Reference

This file documents single-purpose CLI utilities that have been consolidated from individual skills. These are pure command-line tools that wrap APIs or external commands.

**Philosophy:** Simple utilities don't need separate skills. Document them here, execute them directly.

**Model:** Following the `Tools/fabric/` pattern - 242+ Fabric patterns documented as utilities rather than individual skills.

---

## Inference.ts - Unified AI Inference Tool

**Location:** `~/.claude/PAI/Tools/Inference.ts`

Single inference tool with three run levels for different speed/capability trade-offs.

**Usage:**
```bash
# Fast (Haiku) - quick tasks, simple generation
bun ~/.claude/PAI/Tools/Inference.ts --level fast "System prompt" "User prompt"

# Standard (Sonnet) - balanced reasoning, typical analysis
bun ~/.claude/PAI/Tools/Inference.ts --level standard "System prompt" "User prompt"

# Smart (Opus) - deep reasoning, strategic decisions
bun ~/.claude/PAI/Tools/Inference.ts --level smart "System prompt" "User prompt"

# With JSON output
bun ~/.claude/PAI/Tools/Inference.ts --json --level fast "Return JSON" "Input"

# Custom timeout
bun ~/.claude/PAI/Tools/Inference.ts --level standard --timeout 60000 "Prompt" "Input"
```

**Run Levels:**
| Level | Model | Default Timeout | Use Case |
|-------|-------|-----------------|----------|
| **fast** | Haiku | 15s | Quick tasks, simple generation, basic classification |
| **standard** | Sonnet | 30s | Balanced reasoning, typical analysis, decisions |
| **smart** | Opus | 90s | Deep reasoning, strategic decisions, complex analysis |

**Programmatic Usage:**
```typescript
import { inference } from '../PAI/Tools/Inference';

const result = await inference({
  systemPrompt: 'Analyze this',
  userPrompt: 'Content to analyze',
  level: 'standard',  // 'fast' | 'standard' | 'smart'
  expectJson: true,   // optional: parse JSON response
  timeout: 30000,     // optional: custom timeout
});

if (result.success) {
  console.log(result.output);
  console.log(result.parsed);  // if expectJson: true
}
```

**When to Use:**
- "quick inference" → fast
- "analyze this" → standard
- "deep analysis" → smart
- Hooks use this for sentiment analysis, tab titles, work classification

**Technical Details:**
- Uses Claude CLI with subscription (not API key)
- Disables tools and hooks to prevent recursion
- Returns latency metrics for monitoring

---

## RemoveBg.ts - Remove Image Backgrounds

**Location:** `~/.claude/PAI/Tools/RemoveBg.ts`

Remove backgrounds from images using the remove.bg API.

**Usage:**
```bash
# Remove background from single image (overwrites original)
bun ~/.claude/PAI/Tools/RemoveBg.ts /path/to/image.png

# Remove background and save to different path
bun ~/.claude/PAI/Tools/RemoveBg.ts /path/to/input.png /path/to/output.png

# Process multiple images
bun ~/.claude/PAI/Tools/RemoveBg.ts image1.png image2.png image3.png
```

**Environment Variables:**
- `REMOVEBG_API_KEY` - Required for background removal (from `${PAI_DIR}/.env`)

**When to Use:**
- "remove background from this image"
- "remove the background"
- "make this image transparent"

---

## AddBg.ts - Add Background Color

**Location:** `~/.claude/PAI/Tools/AddBg.ts`

Add solid background color to transparent images.

**Usage:**
```bash
# Add specific background color
bun ~/.claude/PAI/Tools/AddBg.ts /path/to/transparent.png "#EAE9DF" /path/to/output.png

# Add brand background color
bun ~/.claude/PAI/Tools/AddBg.ts /path/to/transparent.png --brand /path/to/output.png
```

**When to Use:**
- "add background to this image"
- "create thumbnail with brand background"
- "add the brand color background"

**Brand Color:** `#EAE9DF` (warm paper/sepia tone)

---

## GetTranscript.ts - Extract YouTube Transcripts

**Location:** `~/.claude/PAI/Tools/GetTranscript.ts`

Extract transcripts from YouTube videos using yt-dlp (via fabric).

**Usage:**
```bash
# Extract transcript to stdout
bun ~/.claude/PAI/Tools/GetTranscript.ts "https://www.youtube.com/watch?v=VIDEO_ID"

# Save transcript to file
bun ~/.claude/PAI/Tools/GetTranscript.ts "https://www.youtube.com/watch?v=VIDEO_ID" --save /path/to/transcript.txt
```

**Supported URL Formats:**
- `https://www.youtube.com/watch?v=VIDEO_ID`
- `https://youtu.be/VIDEO_ID`
- `https://www.youtube.com/watch?v=VIDEO_ID&t=123` (with timestamp)
- `https://youtube.com/shorts/VIDEO_ID` (YouTube Shorts)

**When to Use:**
- "get the transcript from this YouTube video"
- "extract transcript from this video"
- "fabric -y <url>" (user explicitly mentions fabric)

**Technical Details:**
- Uses `fabric -y` under the hood
- Prioritizes manual captions when available
- Falls back to auto-generated captions
- Multi-language support (detects automatically)

---

## Voice Server API - Generate Voice Narration

**Location:** Pulse voice compatibility route at `http://localhost:31337/notify`

Send text through Pulse for TTS using the configured backend. Set `voice_enabled:false` for route tests that must not speak.

**Usage:**
```bash
# Single narration segment
curl -X POST http://localhost:31337/notify \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Your text here",
    "voice_id": "main",
    "title": "Voice Narrative",
    "voice_enabled": true
  }'

# Pause between segments
sleep 2
```

**Voice Configuration:**
- **Voice ID:** `main` by default; Pulse resolves backend-specific details
- **Stability:** 0.55 (natural variation in storytelling)
- **Similarity Boost:** 0.85 (maintains authentic sound)
- **Server:** `http://localhost:31337/notify`
- **Max Segment:** 450 characters
- **Pause Between:** 2 seconds

**When to Use:**
- "read this to me"
- "voice narrative"
- "speak this"
- "narrate this"
- "perform this"

**Technical Details:**
- Voice server must be running (`~/.claude/skills/VoiceServer/`)
- Segments longer than 450 chars should be split
- Natural 2-second pauses between segments for storytelling flow
- Uses ElevenLabs API under the hood

---

## extract-transcript.py - Transcribe Audio/Video Files

**Location:** `~/.claude/PAI/Tools/extract-transcript.py`

Local transcription using faster-whisper (4x faster than OpenAI Whisper, 50% less memory). Self-contained UV script for offline transcription.

**Usage:**
```bash
# Transcribe single file (base.en model - recommended)
cd ~/.claude/PAI/Tools/
uv run extract-transcript.py /path/to/audio.m4a

# Use different model
uv run extract-transcript.py audio.m4a --model small.en

# Generate subtitles
uv run extract-transcript.py video.mp4 --format srt

# Batch transcribe folder
uv run extract-transcript.py /path/to/folder/ --batch --model base.en
```

**Supported Formats:**
- **Audio:** m4a, mp3, wav, flac, ogg, aac, wma
- **Video:** mp4, mov, avi, mkv, webm, flv

**Output Formats:**
- **txt** - Plain text transcript (default)
- **json** - Structured JSON with timestamps
- **srt** - SubRip subtitle format
- **vtt** - WebVTT subtitle format

**Model Options:**
| Model | Size | Speed | Accuracy | Use Case |
|-------|------|-------|----------|----------|
| tiny.en | 75MB | Fastest | Basic | Quick drafts, testing |
| **base.en** | 150MB | Fast | Good | **General use (recommended)** |
| small.en | 500MB | Medium | Very Good | Important recordings |
| medium | 1.5GB | Slow | Excellent | Production quality |
| large-v3 | 3GB | Slowest | Best | Critical accuracy needs |

**When to Use:**
- "transcribe this audio"
- "transcribe recording"
- "extract transcript from audio"
- "convert audio to text"
- "generate subtitles"

**Technical Details:**
- 100% local processing (no API calls, completely offline)
- First run auto-installs dependencies via UV (~30 seconds)
- Models auto-download from HuggingFace on first use
- Apple Silicon (M1/M2/M3) optimized
- Processing speed: ~3-5 minutes for 36MB audio file (base.en model)

**Prerequisites:**
- UV package manager: `curl -LsSf https://astral.sh/uv/install.sh | sh`
- No manual model download required (auto-downloads on first use)

---

## YouTubeApi.ts - YouTube Channel & Video Stats

**Location:** `~/.claude/PAI/Tools/YouTubeApi.ts`

Wrapper around YouTube Data API v3 for channel statistics and video metrics.

**Usage:**
```bash
# Get channel statistics
bun ~/.claude/PAI/Tools/YouTubeApi.ts --channel-stats

# Get video statistics
bun ~/.claude/PAI/Tools/YouTubeApi.ts --video-stats VIDEO_ID

# Get latest uploads
bun ~/.claude/PAI/Tools/YouTubeApi.ts --latest-videos
```

**Environment Variables:**
- `YOUTUBE_API_KEY` - Required for API access (from `${PAI_DIR}/.env`)
- `YOUTUBE_CHANNEL_ID` - Default channel ID

**When to Use:**
- "get YouTube stats"
- "YouTube channel statistics"
- "video performance metrics"
- "subscriber count"

**Data Retrieved:**
- Total subscribers
- Total views
- Total videos
- Recent upload performance
- View counts, likes, comments per video

**Technical Details:**
- Uses YouTube Data API v3 REST endpoints
- Quota: 10,000 units per day (free tier)
- Each API call costs ~3-5 quota units

---

## TruffleHog - Scan for Exposed Secrets

**Location:** System-installed CLI tool (`brew install trufflehog`)

Scan directories for 700+ types of credentials and secrets.

**Usage:**
```bash
# Scan directory
trufflehog filesystem /path/to/directory

# Scan git repository
trufflehog git file:///path/to/repo

# Scan with verified findings only
trufflehog filesystem /path/to/directory --only-verified
```

**Installation:**
```bash
brew install trufflehog
```

**When to Use:**
- "check for secrets"
- "scan for sensitive data"
- "find API keys"
- "detect credentials"
- "security audit before commit"

**What It Detects:**
- API keys (OpenAI, AWS, GitHub, Stripe, 700+ services)
- OAuth tokens
- Private keys (SSH, PGP, SSL/TLS)
- Database connection strings
- Passwords in code
- Cloud provider credentials

**Technical Details:**
- Scans files, git history, and commits
- Uses entropy detection + regex patterns
- Verifies findings when possible (calls APIs to check if keys are valid)
- No API key required (standalone CLI tool)

---

## Integration with Other Skills

### Art Skill
- Background removal: `RemoveBg.ts`
- Add backgrounds: `AddBg.ts`

### Blogging Skill
- Image optimization: `RemoveBg.ts`, `AddBg.ts`
- Social preview thumbnails

### Research Skill
- YouTube transcripts: `GetTranscript.ts`
- Audio/video transcription: `extract-transcript.py`
- Voice narration: Voice server API

### Metrics Skill
- YouTube analytics: `YouTubeApi.ts`

### Security Workflows
- Secret scanning: `trufflehog` (system tool)

---

## Adding New Tools

When adding a new utility tool to this system:

1. **Add tool file:** Place `.ts` or `.py` file directly in `~/.claude/PAI/Tools/`
   - Use **Title Case** for filenames (e.g., `GetTranscript.ts`, not `get-transcript.ts`)
   - Keep the directory flat - NO subdirectories

2. **Document here:** Add section to this file with:
   - Tool location (e.g., `~/.claude/PAI/Tools/ToolName.ts`)
   - Usage examples
   - When to use triggers
   - Environment variables (if any)

3. **Update PAI/SKILL.md:** Ensure SYSTEM/TOOLS.md is in the documentation index

4. **Test:** Verify tool works from new location

**Don't create a separate skill** if the entire functionality is just a CLI command with parameters.

---

## Deprecated Skills

The following skills have been consolidated into this Tools system:

- **Images** → `Tools/RemoveBg.ts`, `Tools/AddBg.ts` (2024-12-22)
- **VideoTranscript** → `Tools/GetTranscript.ts` (2024-12-22)
- **VoiceNarration** → Voice server API (2024-12-22)
- **ExtractTranscript** → `Tools/extract-transcript.py`, `Tools/ExtractTranscript.ts` (2024-12-22)
- **YouTube** → `Tools/YouTubeApi.ts` (2024-12-22)
- **Sensitive** → `trufflehog` system tool (2024-12-22)

Archived skill files have been removed.

---

**Last Updated:** 2026-04-01

---

## Cost.ts — Session Analytics

**Location:** `~/.claude/PAI/Tools/Cost.ts`

Reads `MEMORY/LEARNING/REFLECTIONS/algorithm-reflections.jsonl` and produces session cost/quality analytics.

**Usage:**
```bash
# Pretty table output
bun ~/.claude/PAI/Tools/Cost.ts

# Machine-readable JSON
bun ~/.claude/PAI/Tools/Cost.ts --json
```

**Output Includes:**
- Total session count
- Avg sentiment score (1–10)
- Avg ISC pass rate (%)
- Within-budget rate (%)
- Estimated cost by effort tier (heuristic proxy)
- Sessions breakdown by effort level (bar chart)

**When to Use:**
- "how much have I spent on PAI"
- "show session analytics"
- "Algorithm pass rates"

---

## Memory.ts — Memory Database Management

**Location:** `~/.claude/PAI/Tools/Memory.ts`

Manages the unified memory SQLite database at `MEMORY/db/memory.db`. Import legacy memory files, check sync status.

**Usage:**
```bash
# Import legacy memory files into DB
bun ~/.claude/PAI/Tools/Memory.ts import-legacy --source /path/to/old/memory [--dry-run]

# Check DB status and row counts
bun ~/.claude/PAI/Tools/Memory.ts sync-status

# Initialize schema
bun ~/.claude/PAI/Tools/Memory.ts init
```

**Schema:** `memories` table — `id, filename, category, title, body, model_type, created_at, updated_at`

**When to Use:**
- Migrating old memory files into the unified DB
- Checking how many memories are stored
- Diagnosing memory DB state

**Dependencies:** Requires `ulid` package (`bun add ulid` if missing).

---

## MemoryIngest.ts — Session Harvest + DB Sync

**Location:** `~/.claude/PAI/Tools/MemoryIngest.ts`

Runs the session harvester for the latest session, then scans `MEMORY/LEARNING/{ALGORITHM,SYSTEM}/` directories and upserts new `.md` files into the memory DB. Called automatically by `pai.ts` on session exit.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/MemoryIngest.ts
```

**What It Does:**
1. Runs `SessionHarvester.ts --recent 1` to extract latest session data
2. Scans `MEMORY/LEARNING/ALGORITHM/` and `MEMORY/LEARNING/SYSTEM/` for `.md` files
3. Inserts any new files into `memory.db` (skip-if-exists by filename)

**When to Use:**
- Automatically triggered by `pai.ts` post-run
- Manual trigger after bulk memory file additions

---

## distill.ts — Algorithm Reflection Distillation

**Location:** `~/.claude/PAI/Tools/distill.ts`

Reads recent Algorithm reflections (last N days), summarizes them via Ollama (qwen2.5:7b), writes a dated section to `shared-brain/distilled-learnings.md`, and optionally pushes the summary to Open WebUI.

**Usage:**
```bash
# Distill last 7 days (default), write locally + push to OWUI
bun ~/.claude/PAI/Tools/distill.ts

# Look back 14 days
bun ~/.claude/PAI/Tools/distill.ts --days 14

# Preview output without writing or pushing
bun ~/.claude/PAI/Tools/distill.ts --dry-run

# Write locally but skip OWUI push
bun ~/.claude/PAI/Tools/distill.ts --no-push
```

**Output:** Appends to `~/.claude/shared-brain/distilled-learnings.md` (directory auto-created).

**Environment Variables:**
- `OLLAMA_URL` — Ollama server (default: `http://192.168.50.20:11434`)
- `FUNCTIONS_API_URL` — FunctionsAPI base URL (default: `http://localhost:8890`)
- `PAI_API_KEY` — Bearer token for `/v1/owui/push` (see OWUI setup below)

**OWUI_ADMIN_TOKEN Setup:**
The `/v1/owui/push` endpoint and this tool require `OWUI_ADMIN_TOKEN` in `~/.config/PAI/secrets.env`:
```bash
# ~/.config/PAI/secrets.env
OWUI_ADMIN_TOKEN=sk-...   # from Open WebUI Settings → Account → API Keys
```
Generate the token in Open WebUI: **Settings → Account → API Keys → Create new key**.
Without this, OWUI push silently returns 503 (distill.ts local write still succeeds).

**When to Use:**
- "distill recent Algorithm reflections"
- "summarize learning from last week"
- Weekly brain-sync to Open WebUI

---

## gpu_status.sh — GPU Metrics

**Location:** `~/.claude/PAI/Tools/gpu_status.sh`

Queries `nvidia-smi` for GPU name, VRAM total/used, and utilization, returns JSON.

**Usage:**
```bash
bash ~/.claude/PAI/Tools/gpu_status.sh
```

**Output:**
```json
{"status":"OK","gpus":[{"gpu":"NVIDIA RTX...","vram_total_mb":24576,"vram_used_mb":8192,"utilization_percent":45}]}
```

**When to Use:**
- Called by CommandCenter `/api/gpu-status` endpoint
- Manual GPU health check: `bash gpu_status.sh | jq .`
- Returns `{"status":"OK","gpus":[]}` gracefully when `nvidia-smi` is absent

---

## system_health.sh — System Health Metrics

**Location:** `~/.claude/PAI/Tools/system_health.sh`

Reports local host CPU load, RAM total/used as JSON via `top` and `free`.

**Usage:**
```bash
bash ~/.claude/PAI/Tools/system_health.sh
```

**Output:**
```json
{"status":"OK","cpu_load_percent":12.5,"ram_total_mb":64000,"ram_used_mb":18432}
```

**When to Use:**
- Called by CommandCenter `/api/system-health` endpoint (also adds disk stats via `df`)
- Manual system health check: `bash system_health.sh | jq .`

---

## Banner Tools

### Banner.ts — Dynamic Multi-Design Banner

**Location:** `~/.claude/PAI/Tools/Banner.ts`

Randomly selects a banner design based on terminal width. Large terminals (85+ cols) get Navy/Electric/Teal/Ice themes; small terminals get minimal layouts. Run on session start for neofetch-style PAI stats display.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/Banner.ts
```

---

### BannerMatrix.ts — Matrix Digital Rain Banner

**Location:** `~/.claude/PAI/Tools/BannerMatrix.ts`

Green-on-black Katakana rain aesthetic. Neofetch layout with PAI logo emerging from Matrix rain on the left, system stats on the right.

```bash
bun ~/.claude/PAI/Tools/BannerMatrix.ts
```

---

### BannerNeofetch.ts — Modern Neofetch Banner

**Location:** `~/.claude/PAI/Tools/BannerNeofetch.ts`

Blue→purple→cyan gradient theme. Isometric cube left, stats right, sparkline histogram at bottom.

```bash
bun ~/.claude/PAI/Tools/BannerNeofetch.ts
```

---

### BannerRetro.ts — Retro BBS/DOS Banner

**Location:** `~/.claude/PAI/Tools/BannerRetro.ts`

Classic ASCII art aesthetic, amber/green phosphor CRT feel. BBS-style double-line box framing.

```bash
bun ~/.claude/PAI/Tools/BannerRetro.ts
```

---

### BannerTokyo.ts — Tokyo Night Theme Banner

**Location:** `~/.claude/PAI/Tools/BannerTokyo.ts`

Deep blue-black with soft neon accents (#7aa2f7 blue, #bb9af7 magenta, #9ece6a green). Static banner, no system stats.

```bash
bun ~/.claude/PAI/Tools/BannerTokyo.ts
```

---

### BannerPrototypes.ts — Banner Design Prototypes

**Location:** `~/.claude/PAI/Tools/BannerPrototypes.ts`

Test bed for banner designs (Glitch Cyberpunk, etc.). Not meant for production use — run to preview prototype designs during banner development.

```bash
bun ~/.claude/PAI/Tools/BannerPrototypes.ts
```

---

### NeofetchBanner.ts — Cyberpunk Neofetch Banner

**Location:** `~/.claude/PAI/Tools/NeofetchBanner.ts`

Cyberpunk/hacker aesthetic with Tokyo Night colors, hex addresses, binary streams, targeting reticle elements. Alternative to Banner.ts.

```bash
bun ~/.claude/PAI/Tools/NeofetchBanner.ts
```

---

### PAILogo.ts — PAI ASCII Logo

**Location:** `~/.claude/PAI/Tools/PAILogo.ts`

Renders the PAI logo (figlet-style A+I where P is hidden in color). Purple = P portion, blue = A right leg, cyan = I.

```bash
bun ~/.claude/PAI/Tools/PAILogo.ts
```

---

## Budget & Token Routing Tools

### BudgetReport.ts — Token Budget CLI

**Location:** `~/.claude/PAI/Tools/BudgetReport.ts`

Reports Claude's 5-hour rolling token window. Weekly/monthly views are cost-awareness only.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/BudgetReport.ts            # show budget status
bun ~/.claude/PAI/Tools/BudgetReport.ts --set-window-limit 50000  # calibrate after rate-limit
```

**When to Use:**
- Algorithm OBSERVE phase budget preflight
- "how much budget do I have left"

---

### budget-shared.ts — Budget Tracking Shared Library

**Location:** `~/.claude/PAI/Tools/budget-shared.ts`

Shared types and utilities imported by `BudgetTracker.hook.ts` (writer) and `BudgetReport.ts` (reader). Not a standalone CLI — import as a module.

```typescript
import { loadDeduplicatedRecords, windowOutputTokens } from '../PAI/Tools/budget-shared';
```

---

### ModelProfile.ts — Model Profile Manager

**Location:** `~/.claude/PAI/Tools/ModelProfile.ts`

Sets routing aggressiveness: `quality` (Claude always), `balanced` (default), `budget` (Ollama preferred).

**Usage:**
```bash
bun ~/.claude/PAI/Tools/ModelProfile.ts              # show current profile
bun ~/.claude/PAI/Tools/ModelProfile.ts set budget   # route to Ollama when possible
bun ~/.claude/PAI/Tools/ModelProfile.ts set quality  # always use Claude
bun ~/.claude/PAI/Tools/ModelProfile.ts list
```

---

### ModelRouter.ts — Ollama Model Router

**Location:** `~/.claude/PAI/Tools/ModelRouter.ts`

Maps task descriptions to the correct Ollama model via keyword routing table. Import as library or use CLI.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/ModelRouter.ts "score these CVEs for CVSS"
# → qwen2.5:7b
```

```typescript
import { routeModel } from '../PAI/Tools/ModelRouter';
const model = routeModel(taskDescription);
```

| Output | Use Case |
|--------|----------|
| `llama3.2:3b` | Classification, sentiment, labeling |
| `qwen2.5:7b` | Structured JSON, scoring |
| `gemma2:9b` | Summarization, doc analysis |
| `deepseek-r1:7b` | Multi-step reasoning |
| `nemotron-mini:latest` | Default structured tasks |

---

### OllamaMode.ts — Ollama Routing Mode

**Location:** `~/.claude/PAI/Tools/OllamaMode.ts`

Controls when PAI routes Task spawns to Ollama instead of Claude.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/OllamaMode.ts              # show status
bun ~/.claude/PAI/Tools/OllamaMode.ts max          # route ALL spawns to Ollama
bun ~/.claude/PAI/Tools/OllamaMode.ts prefer       # route WARN+ spawns to Ollama
bun ~/.claude/PAI/Tools/OllamaMode.ts failover     # Ollama only at CRITICAL (default)
bun ~/.claude/PAI/Tools/OllamaMode.ts off          # never use Ollama
bun ~/.claude/PAI/Tools/OllamaMode.ts model qwen2.5:7b
```

**When to Use:**
- "use Ollama for everything" → `max`
- "save budget" → `prefer`
- Debugging routing issues

---

### Scheduler.ts — Compute Scheduler

**Location:** `~/.claude/PAI/Tools/Scheduler.ts`

Routes tasks based on GPU VRAM availability: heavy tasks → local Ollama if VRAM >4GB, else Mercury/Gemini CLI. Fallback chain: Ollama → Mercury → Gemini → Claude.

```typescript
import { routeTask, getVramUsage } from '../PAI/Tools/Scheduler';
```

---

## Memory & Database Tools

### harvest.ts — PAI Content Harvester

**Location:** `~/.claude/PAI/Tools/harvest.ts`

Walks `/PAI`, classifies files via Ollama, routes into unified DB + research files + review queue.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/harvest.ts                 # harvest (limit 100 files)
bun ~/.claude/PAI/Tools/harvest.ts --dry-run       # preview without writing
bun ~/.claude/PAI/Tools/harvest.ts --limit 500     # process more files
bun ~/.claude/PAI/Tools/harvest.ts --no-ollama     # heuristic-only classification
bun ~/.claude/PAI/Tools/harvest.ts --reset         # re-process everything
bun ~/.claude/PAI/Tools/harvest.ts --dir Tools     # restrict to /PAI/Tools
```

---

### google-apps-ingest.ts — Google Takeout Ingest

**Location:** `~/.claude/PAI/Tools/google-apps-ingest.ts`

Ingests Google Takeout export (Drive, Calendar, Keep, Location) into the documents table.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/google-apps-ingest.ts --takeout ~/Downloads/Takeout
bun ~/.claude/PAI/Tools/google-apps-ingest.ts --takeout ~/Downloads/Takeout --source drive,calendar
bun ~/.claude/PAI/Tools/google-apps-ingest.ts --dry-run --limit 100
```

---

### substrate-ingest.ts — Substrate Knowledge Ingest

**Location:** `~/.claude/PAI/Tools/substrate-ingest.ts`

Ingests Substrate repository knowledge components (Arguments, Claims, Experiments, Frames, etc.) into the documents table.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/substrate-ingest.ts --substrate /path/to/substrate
bun ~/.claude/PAI/Tools/substrate-ingest.ts --dry-run --limit 100
```

---

### youtube-ingest.ts — YouTube Watch History Ingest

**Location:** `~/.claude/PAI/Tools/youtube-ingest.ts`

Ingests YouTube watch history and playlists from Google Takeout or yt-dlp into the documents table.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/youtube-ingest.ts --takeout ~/Downloads/Takeout
bun ~/.claude/PAI/Tools/youtube-ingest.ts --playlist "https://youtube.com/playlist?list=..."
bun ~/.claude/PAI/Tools/youtube-ingest.ts --playlist-file /tmp/playlist.jsonl
bun ~/.claude/PAI/Tools/youtube-ingest.ts --dry-run --limit 50
```

---

### purge-harvest-noise.ts — Harvest Noise Cleanup

**Location:** `~/.claude/PAI/Tools/purge-harvest-noise.ts`

One-time cleanup: removes `.venv`, `site-packages`, `dist-info`, `_ARCHIVE` paths from the `pai-harvest` category in the documents DB.

```bash
bun ~/.claude/PAI/Tools/purge-harvest-noise.ts
```

---

### MemoryConflictResolver.ts — ICM Deduplication

**Location:** `~/.claude/PAI/Tools/MemoryConflictResolver.ts`

Identifies ICM memories with the same topic that likely conflict (two contradicting "preferences" entries, etc.). Prints `icm_memory_forget` commands to remove stale entries — does not auto-delete.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/MemoryConflictResolver.ts
bun ~/.claude/PAI/Tools/MemoryConflictResolver.ts --topic preferences
bun ~/.claude/PAI/Tools/MemoryConflictResolver.ts --dry-run
```

---

### MemoryStatus.ts — Memory DB Health

**Location:** `~/.claude/PAI/Tools/MemoryStatus.ts`

Reports PAI unified memory status: distillation progress, MEMORY/WORK count, algorithm reflections count, distiller log.

```bash
bun ~/.claude/PAI/Tools/MemoryStatus.ts
```

---

### merge-m710q-databases.ts — m710q Backup Merge

**Location:** `~/.claude/PAI/Tools/merge-m710q-databases.ts`

Merges m710q backup SQLite (`/share/PAI/Memory/pai-claude-backup/STATE/pai-memory.db`) into local ICM via `icm store` CLI.

```bash
bun ~/.claude/PAI/Tools/merge-m710q-databases.ts
bun ~/.claude/PAI/Tools/merge-m710q-databases.ts --dry-run
```

---

### process-m710q-memory.ts — m710q Memory Files to ICM

**Location:** `~/.claude/PAI/Tools/process-m710q-memory.ts`

Processes m710q `antigravity/` and `SECURITY/` memory files into ICM topics.

```bash
bun ~/.claude/PAI/Tools/process-m710q-memory.ts
bun ~/.claude/PAI/Tools/process-m710q-memory.ts --dry-run --verbose
```

---

### Batch2Distiller.ts — WORK PRD Insight Distiller

**Location:** `~/.claude/PAI/Tools/Batch2Distiller.ts`

Reads MEMORY/WORK PRDs, summarizes decisions/patterns via Ollama (qwen2.5:7b), maintains idempotent state file to avoid re-processing.

```bash
bun ~/.claude/PAI/Tools/Batch2Distiller.ts
```

---

### HistoricalDistiller.ts — Legacy Postgres Distiller

**Location:** `~/.claude/PAI/Tools/HistoricalDistiller.ts`

Older variant that wrote to Postgres (`pai_memory` DB on localhost:5433) with embeddings. Superseded by Batch2Distiller.ts and ICM for most use cases.

---

## Session & Algorithm Tools

### algorithm.ts — Algorithm CLI Runner

**Location:** `~/.claude/PAI/Tools/algorithm.ts`

Unified CLI for executing Algorithm sessions against PRDs. Two modes: `loop` (autonomous iteration via SDK, no human) and `interactive` (launches full Claude session with PRD context).

**Usage:**
```bash
bun ~/.claude/PAI/Tools/algorithm.ts loop --prd MEMORY/WORK/slug/PRD.md
bun ~/.claude/PAI/Tools/algorithm.ts interactive --prd MEMORY/WORK/slug/PRD.md
```

Dashboard integration: creates persistent state in `MEMORY/STATE/algorithms/`, registers in `session-names.json`.

---

### AlgorithmPhaseReport.ts — Algorithm State Writer

**Location:** `~/.claude/PAI/Tools/AlgorithmPhaseReport.ts`

Writes algorithm phase/criterion/agent/capability state to `MEMORY/STATE/algorithm-phase.json` for CommandCenter dashboard display.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/AlgorithmPhaseReport.ts phase --phase OBSERVE --task "Auth rebuild" --sla Standard
bun ~/.claude/PAI/Tools/AlgorithmPhaseReport.ts criterion --id 1 --desc "JWT rejects expired" --status pending
bun ~/.claude/PAI/Tools/AlgorithmPhaseReport.ts criterion --id 1 --status completed --evidence "Tests pass"
bun ~/.claude/PAI/Tools/AlgorithmPhaseReport.ts agent --name engineer-1 --type Engineer --status active
bun ~/.claude/PAI/Tools/AlgorithmPhaseReport.ts capabilities --list "Task Tool,Engineer Agents"
```

---

### ActivityParser.ts — Session Activity Parser

**Location:** `~/.claude/PAI/Tools/ActivityParser.ts`

Parses session activity for PAI repo update documentation. Writes dated entries to `MEMORY/PAISYSTEMUPDATES/`.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/ActivityParser.ts --today
bun ~/.claude/PAI/Tools/ActivityParser.ts --today --generate
bun ~/.claude/PAI/Tools/ActivityParser.ts --session abc-123
```

---

### SessionHarvester.ts — Session Learning Extractor

**Location:** `~/.claude/PAI/Tools/SessionHarvester.ts`

Harvests insights from `~/.claude/projects/` session transcripts and writes structured learnings to `LEARNING/`.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/SessionHarvester.ts --recent 5
bun ~/.claude/PAI/Tools/SessionHarvester.ts --all --dry-run
bun ~/.claude/PAI/Tools/SessionHarvester.ts --session abc-123
```

---

### SessionProgress.ts — Multi-Session Continuity

**Location:** `~/.claude/PAI/Tools/SessionProgress.ts`

Manages session continuity files (decisions, work items, context) for multi-session tasks. Based on Anthropic's claude-progress.txt pattern.

```bash
bun ~/.claude/PAI/Tools/SessionProgress.ts <command> [options]
```

---

### SessionReport.ts — End-of-Session Summary

**Location:** `~/.claude/PAI/Tools/SessionReport.ts`

Generates structured session summary: work done, ISC completed, decisions, seeds planted, git activity. Triggered by Stop hook or manually.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/SessionReport.ts            # report for today
bun ~/.claude/PAI/Tools/SessionReport.ts --save     # save to MEMORY/STATE/
bun ~/.claude/PAI/Tools/SessionReport.ts --hook     # quiet if no active work
```

---

### FailureCapture.ts — Failure Analysis System

**Location:** `~/.claude/PAI/Tools/FailureCapture.ts`

Creates comprehensive context dumps for low-sentiment events (ratings 1-3). Writes `CONTEXT.md`, `transcript.jsonl`, and `sentiment.json` to `MEMORY/LEARNING/FAILURES/<YYYY-MM>/`.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/FailureCapture.ts <transcript_path> <rating> "summary" ["detailed context"]
```

```typescript
import { captureFailure } from './FailureCapture';
await captureFailure({ transcriptPath, rating, sentimentSummary });
```

---

### LearningPatternSynthesis.ts — Rating Pattern Analyzer

**Location:** `~/.claude/PAI/Tools/LearningPatternSynthesis.ts`

Aggregates `LEARNING/SIGNALS/ratings.jsonl` to find recurring patterns and generates synthesis reports.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/LearningPatternSynthesis.ts --week
bun ~/.claude/PAI/Tools/LearningPatternSynthesis.ts --month --dry-run
bun ~/.claude/PAI/Tools/LearningPatternSynthesis.ts --all
```

---

### ReflectionHealth.ts — Reflection JSONL Validator

**Location:** `~/.claude/PAI/Tools/ReflectionHealth.ts`

Checks `algorithm-reflections.jsonl` for empty/malformed lines, reports counts, optionally repairs.

```bash
bun ~/.claude/PAI/Tools/ReflectionHealth.ts
```

---

### ReflectionReview.ts — Reflection Pattern Reviewer

**Location:** `~/.claude/PAI/Tools/ReflectionReview.ts`

Clusters algorithm reflections into recurring failure patterns. Supports deferral and dismiss.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/ReflectionReview.ts --last 20
bun ~/.claude/PAI/Tools/ReflectionReview.ts --defer "sequential network calls"
bun ~/.claude/PAI/Tools/ReflectionReview.ts --dismiss "pattern name"
bun ~/.claude/PAI/Tools/ReflectionReview.ts --list-deferrals
```

**When to Use:**
- After Algorithm LEARN phase rating
- "show recurring failures"
- Weekly pattern review

---

## Agent Coordination Tools

### AgentLedger.ts — Agent Balance Ledger

**Location:** `~/.claude/PAI/Tools/AgentLedger.ts`

Tracks agent "credit" balances in `MEMORY/STATE/agent-ledger.json`. Agents earn/spend credits via deposits/withdrawals. Used by PoW system to throttle agent-to-agent loops.

```typescript
import { AgentLedger } from './AgentLedger';
AgentLedger.deposit('agent-id', 1.0, 'completed task');
AgentLedger.withdraw('agent-id', 0.5, 'sent message');
const balance = AgentLedger.getBalance('agent-id');
```

---

### AgentPoW.ts — Agent Proof-of-Work Library

**Location:** `~/.claude/PAI/Tools/AgentPoW.ts`

Hashcash-style SHA-256 puzzles for throttling agent-to-agent loops. Default difficulty: 4 leading zero hex digits.

```typescript
import { AgentPoW } from './AgentPoW';
const challenge = AgentPoW.generateChallenge('recipient-agent-id');
const nonce = AgentPoW.solveHashcash(challenge, 'sender-id');
const valid = AgentPoW.verifyPoW({ nonce, challenge, sender_id: 'sender-id' });
```

---

### InboxWatchdog.ts — Agent Inbox Monitor

**Location:** `~/.claude/PAI/Tools/InboxWatchdog.ts`

Watches `MEMORY/STATE/agent-inbox/{gemini,claude,antigravity,ollama}/` directories. Notifies via voice when new task files arrive. Long-running daemon.

```bash
bun ~/.claude/PAI/Tools/InboxWatchdog.ts
```

---

### SwitchboardClient.ts — Inter-Agent WebSocket Client

**Location:** `~/.claude/PAI/Tools/SwitchboardClient.ts`

Real-time inter-agent communication via WebSockets. Enforces channel allowlist (delegation:, broadcast:all, status:).

**Usage:**
```bash
bun ~/.claude/PAI/Tools/SwitchboardClient.ts --channel delegation:gemini --subscribe
bun ~/.claude/PAI/Tools/SwitchboardClient.ts --channel delegation:gemini --publish '{"task":"scan"}'
```

---

### OpaiQueue.ts — OpenCode Task Queue

**Location:** `~/.claude/PAI/Tools/OpaiQueue.ts`

Manages opai (OpenCode) task concurrency, TTL, dead-letter, and deferred tasks.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/OpaiQueue.ts status
bun ~/.claude/PAI/Tools/OpaiQueue.ts enqueue "analyze security PRD"
bun ~/.claude/PAI/Tools/OpaiQueue.ts dequeue
bun ~/.claude/PAI/Tools/OpaiQueue.ts complete
bun ~/.claude/PAI/Tools/OpaiQueue.ts dead-letter
```

---

### OpaiDispatch.ts — OpenCode Refusal Router

**Location:** `~/.claude/PAI/Tools/OpaiDispatch.ts`

Routes opai task refusals to alternate agents or logs for Duane. Tracks refusals in `MEMORY/STATE/opai-refusals.jsonl`.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/OpaiDispatch.ts refuse "task title" --reason capability --context "hit error X"
bun ~/.claude/PAI/Tools/OpaiDispatch.ts list
bun ~/.claude/PAI/Tools/OpaiDispatch.ts route <id>
bun ~/.claude/PAI/Tools/OpaiDispatch.ts dismiss <id>
```

---

### CouncilSession.ts — Council Session Tracker

**Location:** `~/.claude/PAI/Tools/CouncilSession.ts`

JSONL-based session and claim tracking for UnifiedCouncil. Handles contradiction detection and domain strength matrix updates.

```typescript
import { createSession, closeSession, recordClaim } from '../PAI/Tools/CouncilSession';
const session = createSession('prompt text');
closeSession(session.session_id, 'complete');
```

---

## Security & Audit Tools

### Audit.ts — Structured Event Logger

**Location:** `~/.claude/PAI/Tools/Audit.ts`

Appends structured audit events to `MEMORY/RAW/` for Loki/Promtail ingestion.

```typescript
import { logAudit } from './Audit';
logAudit({ level: 'info', component: 'MyTool', msg: 'Started', details: { key: 'val' } });
```

---

### Security.ts — Progressive Disclosure Permission Model

**Location:** `~/.claude/PAI/Tools/Security.ts`

Enforces PAI permission model: READ/WRITE/BASH/DESTRUCTIVE operations with confirmation delays for destructive actions. Library — not a CLI.

```typescript
import { checkSecurity } from './Security';
const result = await checkSecurity({ op: 'DESTRUCTIVE', resource: '/data', agent: 'claude' });
if (!result.allowed) throw new Error(result.reason);
```

---

### jit-cred.ts — Just-In-Time Credential Fetcher

**Location:** `~/.claude/PAI/Tools/jit-cred.ts`

Fetches credentials at use-time from `pass` store or env vars. All accesses are logged. Never holds long-lived static credentials.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/jit-cred.ts ANTHROPIC_API_KEY
bun ~/.claude/PAI/Tools/jit-cred.ts OPENAI_API_KEY
bun ~/.claude/PAI/Tools/jit-cred.ts github/token
```

Backends (in order): `pass` store → env vars (with deprecation warning).

---

### fetch-secret.ts — Vaultwarden Secret Retrieval

**Location:** `~/.claude/PAI/Tools/fetch-secret.ts`

Retrieves secrets from Vaultwarden API at `http://192.168.50.6:8180`. Requires `BW_SESSION` env var.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/fetch-secret.ts <AGENT_ID> <SECRET_NAME>
```

---

### RBACSecurityAudit.ts — RBAC Enforcement Verifier

**Location:** `~/.claude/PAI/Tools/RBACSecurityAudit.ts`

Tests that `fetch-secret.sh` RBAC is correctly enforced: unknown agents denied, standard agents denied all secrets, privileged agents limited to allowlist, admin agents allowed.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/RBACSecurityAudit.ts           # dry-run (no vault hit)
bun ~/.claude/PAI/Tools/RBACSecurityAudit.ts --live    # attempt real retrieval
```

---

### SecretScan.ts — Secret Scanner CLI

**Location:** `~/.claude/PAI/Tools/SecretScan.ts`

TruffleHog wrapper. Scans directories for 700+ credential types.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/SecretScan.ts /path/to/dir
bun ~/.claude/PAI/Tools/SecretScan.ts . --verbose
bun ~/.claude/PAI/Tools/SecretScan.ts . --json --verify
```

---

### SystemAudit.ts — Automated Weekly Auditor

**Location:** `~/.claude/PAI/Tools/SystemAudit.ts`

Checks hostname, uptime, NAS storage mounts, updates the assets DB table. Run weekly via cron.

```bash
bun ~/.claude/PAI/Tools/SystemAudit.ts
```

---

### UpgradeCheck.ts — Pre-Upgrade Patch Classifier

**Location:** `~/.claude/PAI/Tools/UpgradeCheck.ts`

Before `git pull` of upstream PAI changes, classifies local patches as SAFE / CONFLICT / RETIRE by checking if upstream changed the same files.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/UpgradeCheck.ts               # classify all patches
bun ~/.claude/PAI/Tools/UpgradeCheck.ts --patch 3     # check specific patch
bun ~/.claude/PAI/Tools/UpgradeCheck.ts --upstream    # fetch upstream diff first
```

---

## PAI System Management Tools

### pai.ts — Personal AI CLI Launcher

**Location:** `~/.claude/PAI/Tools/pai.ts`

Comprehensive CLI for launching Claude Code with dynamic MCP loading, profile management, and version checking.

**Usage:**
```bash
pai                    # Launch Claude (default profile)
pai -m bd              # Launch with Bright Data MCP
pai -r / --resume      # Resume last session
pai --local            # Stay in current directory
pai update             # Update Claude Code
pai profiles           # List available profiles
pai mcp list           # List available MCPs
```

---

### BuildCLAUDE.ts — CLAUDE.md Generator

**Location:** `~/.claude/PAI/Tools/BuildCLAUDE.ts`

Reads `CLAUDE.md.template`, resolves variables from `settings.json` and `PAI/Algorithm/LATEST`, writes `CLAUDE.md`. Called by SessionStart hook automatically.

```bash
bun ~/.claude/PAI/Tools/BuildCLAUDE.ts
```

---

### GenerateHandoff.ts — Handoff File Writer

**Location:** `~/.claude/PAI/Tools/GenerateHandoff.ts`

Writes `HANDOFF.md` to each active project's sovereign-data dir and to `~/.claude/MEMORY/HANDOFF.md`. Probes sovereign stack services. Called by GenerateHandoff hook on every session Stop.

```bash
bun ~/.claude/PAI/Tools/GenerateHandoff.ts
```

---

### GenerateSkillIndex.ts — Skill Index Builder

**Location:** `~/.claude/PAI/Tools/GenerateSkillIndex.ts`

Parses all `SKILL.md` files under `~/.claude/skills/` and builds `skills/skill-index.json` for dynamic skill discovery by the Algorithm.

```bash
bun ~/.claude/PAI/Tools/GenerateSkillIndex.ts
```

**When to Use:**
- After adding or modifying any skill
- "rebuild skill index"

---

### ValidateSkillStructure.ts — Skill Validator

**Location:** `~/.claude/PAI/Tools/ValidateSkillStructure.ts`

Validates skill directory structure: SKILL.md frontmatter, "USE WHEN" trigger presence, duplicate name detection, symlink cycle detection.

```bash
bun ~/.claude/PAI/Tools/ValidateSkillStructure.ts
```

---

### RebuildPAI.ts — PAI SKILL.md Assembler

**Location:** `~/.claude/PAI/Tools/RebuildPAI.ts`

Assembles `SKILL.md` from numbered component files in `skills/PAI/Components/`. Run after editing component files.

```bash
bun ~/.claude/PAI/Tools/RebuildPAI.ts
```

---

### GetCounts.ts — PAI System Counts

**Location:** `~/.claude/PAI/Tools/GetCounts.ts`

Single source of truth for PAI system counts (skills, workflows, hooks, signals, files, work, research). Both `Banner.ts` and `statusline-command.sh` use this for consistent numbers everywhere.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/GetCounts.ts            # JSON output
bun ~/.claude/PAI/Tools/GetCounts.ts --plain    # human-readable
```

---

### AssetInventory.ts — Infrastructure Asset Registry

**Location:** `~/.claude/PAI/Tools/AssetInventory.ts`

Maintains inventory of network assets and services. Syncs from Tailscale API, HTTP-probes services.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/AssetInventory.ts list
bun ~/.claude/PAI/Tools/AssetInventory.ts seed       # seed known devices from static data
bun ~/.claude/PAI/Tools/AssetInventory.ts sync       # sync from Tailscale API
bun ~/.claude/PAI/Tools/AssetInventory.ts probe      # HTTP-probe services, update status
bun ~/.claude/PAI/Tools/AssetInventory.ts dns        # print DNS tie-in table
```

---

### FeatureRegistry.ts — Feature Tracking CLI

**Location:** `~/.claude/PAI/Tools/FeatureRegistry.ts`

JSON-based feature tracking for complex multi-feature tasks. More robust than Markdown (models less likely to corrupt structured data).

**Usage:**
```bash
bun ~/.claude/PAI/Tools/FeatureRegistry.ts init <project>
bun ~/.claude/PAI/Tools/FeatureRegistry.ts add <project> <feature>
bun ~/.claude/PAI/Tools/FeatureRegistry.ts update <project> <id>
bun ~/.claude/PAI/Tools/FeatureRegistry.ts list <project>
bun ~/.claude/PAI/Tools/FeatureRegistry.ts next <project>
```

---

### PlanChecker.ts — PRD Plan Validator

**Location:** `~/.claude/PAI/Tools/PlanChecker.ts`

Uses Ollama `llama3.2:3b` to verify ISC criteria are atomic, complete, and sufficient. Run after PLAN phase, before EXECUTE.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/PlanChecker.ts                  # check most recent PRD
bun ~/.claude/PAI/Tools/PlanChecker.ts <slug-or-path>   # specific PRD
bun ~/.claude/PAI/Tools/PlanChecker.ts --list           # list recent PRDs
```

Exit codes: 0=pass, 1=fail, 2=error.

---

### PeerReview.ts — Agent Activity Peer Review

**Location:** `~/.claude/PAI/Tools/PeerReview.ts`

Reads recent MEMORY/WORK PRDs + work.json, queries Ollama `deepseek-r1:7b` for gap analysis, writes findings to `MEMORY/STATE/agent-review-queue.jsonl`.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/PeerReview.ts              # run review
bun ~/.claude/PAI/Tools/PeerReview.ts --show       # show pending queue
bun ~/.claude/PAI/Tools/PeerReview.ts --dismiss <id>
bun ~/.claude/PAI/Tools/PeerReview.ts --days 14
```

---

### MetricsExporter.ts — Metrics to VictoriaMetrics

**Location:** `~/.claude/PAI/Tools/MetricsExporter.ts`

Pushes PAI session metrics (budget, counts, queue states) to VictoriaMetrics at `http://localhost:8428/api/v1/import/prometheus`. View in Grafana at `http://localhost:3001`.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/MetricsExporter.ts
bun ~/.claude/PAI/Tools/MetricsExporter.ts --dry-run
```

---

### OwuiSync.ts — Open WebUI Knowledge Sync

**Location:** `~/.claude/PAI/Tools/OwuiSync.ts`

Pushes selected PAI files to Open WebUI knowledge base for RAG pipeline access.

**Prerequisites:** `OWUI_API_KEY` and `OWUI_KNOWLEDGE_ID` in `~/.config/PAI/secrets.env`.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/OwuiSync.ts               # dry-run
bun ~/.claude/PAI/Tools/OwuiSync.ts --push        # push to OWUI
bun ~/.claude/PAI/Tools/OwuiSync.ts --list-kb     # list knowledge bases
```

---

## Wisdom & Learning Tools

### WisdomDomainClassifier.ts — Wisdom Frame Router

**Location:** `~/.claude/PAI/Tools/WisdomDomainClassifier.ts`

Keyword-based classifier that maps request content to `MEMORY/WISDOM/FRAMES/{domain}.md` files.

**Usage:**
```bash
echo "deploy the worker" | bun ~/.claude/PAI/Tools/WisdomDomainClassifier.ts
bun ~/.claude/PAI/Tools/WisdomDomainClassifier.ts --text "fix the login bug"
bun ~/.claude/PAI/Tools/WisdomDomainClassifier.ts --list
```

Output: JSON array of `{ domain, path, relevance }` objects.

---

### WisdomFrameUpdater.ts — Wisdom Frame Writer

**Location:** `~/.claude/PAI/Tools/WisdomFrameUpdater.ts`

Adds new observations to `MEMORY/WISDOM/FRAMES/{domain}.md`: principles, contextual rules, predictions, anti-patterns, evolution logs.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/WisdomFrameUpdater.ts --domain communication --observation "User preferred bullet points"
bun ~/.claude/PAI/Tools/WisdomFrameUpdater.ts --domain development --observation "Refactoring without permission caused pushback" --type anti-pattern
bun ~/.claude/PAI/Tools/WisdomFrameUpdater.ts --from-session
```

---

### WisdomCrossFrameSynthesizer.ts — Cross-Frame Synthesis

**Location:** `~/.claude/PAI/Tools/WisdomCrossFrameSynthesizer.ts`

Extracts shared principles that appear across 2+ wisdom domains and writes verified cross-domain principles to `WISDOM/PRINCIPLES/verified.md`. Run weekly.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/WisdomCrossFrameSynthesizer.ts
bun ~/.claude/PAI/Tools/WisdomCrossFrameSynthesizer.ts --dry-run
bun ~/.claude/PAI/Tools/WisdomCrossFrameSynthesizer.ts --health
```

---

### SkillGapDetector.ts — Skill Gap Miner

**Location:** `~/.claude/PAI/Tools/SkillGapDetector.ts`

Mines `reflection_q3` fields from algorithm reflections to find systematic skill gaps. Catalog entries with no matching skill are prime formalization candidates.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/SkillGapDetector.ts              # full report
bun ~/.claude/PAI/Tools/SkillGapDetector.ts --json
bun ~/.claude/PAI/Tools/SkillGapDetector.ts --top 5
bun ~/.claude/PAI/Tools/SkillGapDetector.ts --link       # inject [[wikilinks]] into recent PRDs
```

---

### theme-extractor.ts — Improvement Theme Extractor

**Location:** `~/.claude/PAI/Tools/theme-extractor.ts`

Surfaces PAI improvement themes via four methods: structural (directory traverse), slugs (PRD frequency analysis), reflections (failure theme extraction), hooks (lifecycle coverage gaps).

**Usage:**
```bash
bun ~/.claude/PAI/Tools/theme-extractor.ts                   # all methods
bun ~/.claude/PAI/Tools/theme-extractor.ts --output-catalog  # write catalog entries
bun ~/.claude/PAI/Tools/theme-extractor.ts --method slugs    # single method
```

---

### Seeds.ts — Future Idea Tracker

**Location:** `~/.claude/PAI/Tools/Seeds.ts`

Plant forward-looking ideas during work without derailing context. Seeds surface at session start via `LoadContext.hook.ts`.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/Seeds.ts plant "add dark mode toggle" --milestone "UI polish"
bun ~/.claude/PAI/Tools/Seeds.ts list
bun ~/.claude/PAI/Tools/Seeds.ts surface        # show seeds relevant to current work
bun ~/.claude/PAI/Tools/Seeds.ts done <id>
bun ~/.claude/PAI/Tools/Seeds.ts dismiss <id>
```

---

### Workstreams.ts — Parallel Milestone Tracks

**Location:** `~/.claude/PAI/Tools/Workstreams.ts`

Tracks parallel work tracks (each with PRD slug, git branch, status). Active workstream surfaced by `LoadContext.hook.ts` at session start.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/Workstreams.ts list
bun ~/.claude/PAI/Tools/Workstreams.ts create "feature-x" --branch feat/x --prd 20260419-feature-x
bun ~/.claude/PAI/Tools/Workstreams.ts switch "feature-x"
bun ~/.claude/PAI/Tools/Workstreams.ts complete "feature-x"
bun ~/.claude/PAI/Tools/Workstreams.ts status
```

---

### OpinionTracker.ts — Confidence-Based Opinion Tracker

**Location:** `~/.claude/PAI/Tools/OpinionTracker.ts`

Tracks beliefs about working with Duane with confidence scores. Evidence accumulates from sessions; opinions evolve.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/OpinionTracker.ts add "prefers concise responses" --category communication
bun ~/.claude/PAI/Tools/OpinionTracker.ts evidence "prefers concise" --supporting "Got positive reaction"
bun ~/.claude/PAI/Tools/OpinionTracker.ts list
bun ~/.claude/PAI/Tools/OpinionTracker.ts show "prefers concise responses"
```

Confidence rules: +0.02 per supporting instance, -0.05 per counter, +0.10 explicit confirmation, -0.20 explicit contradiction.

---

### RelationshipReflect.ts — Relationship Evolution

**Location:** `~/.claude/PAI/Tools/RelationshipReflect.ts`

Periodic reflection (daily or on-demand) that evolves `MEMORY/RELATIONSHIP/` files: updates opinion confidence, scans for milestones, checks patterns.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/RelationshipReflect.ts
bun ~/.claude/PAI/Tools/RelationshipReflect.ts --opinions-only
bun ~/.claude/PAI/Tools/RelationshipReflect.ts --milestones-only
bun ~/.claude/PAI/Tools/RelationshipReflect.ts --dry-run
```

---

### MorningRitual.ts — ADHD-Optimized Daily Check-In

**Location:** `~/.claude/PAI/Tools/MorningRitual.ts`

Reads memory DB for open loops, tasks, and PRDs. Provides focused morning review without context overload.

```bash
bun ~/.claude/PAI/Tools/MorningRitual.ts
```

---

## Pipeline Tools

### PipelineMonitor.ts — Real-Time Pipeline Monitor

**Location:** `~/.claude/PAI/Tools/PipelineMonitor.ts`

WebSocket server + UI for monitoring pipeline execution across multiple agents in real-time.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/PipelineMonitor.ts               # start on port 8765
bun ~/.claude/PAI/Tools/PipelineMonitor.ts --port=8766
# Open http://localhost:8765 for UI
```

---

### PipelineOrchestrator.ts — Pipeline Runner

**Location:** `~/.claude/PAI/Tools/PipelineOrchestrator.ts`

Runs YAML-defined pipelines and reports progress to PipelineMonitor.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/PipelineOrchestrator.ts run <pipeline> --input '{"key":"val"}' --agent "engineer-1"
bun ~/.claude/PAI/Tools/PipelineOrchestrator.ts demo
```

---

## Transcript & Audio Tools

### TranscriptParser.ts — Transcript Parsing Library

**Location:** `~/.claude/PAI/Tools/TranscriptParser.ts`

Shared library for extracting content from Claude Code `.jsonl` transcript files. Used by Stop hooks for voice, tab state, and response capture.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/TranscriptParser.ts <transcript_path>
bun ~/.claude/PAI/Tools/TranscriptParser.ts <transcript_path> --voice
bun ~/.claude/PAI/Tools/TranscriptParser.ts <transcript_path> --state
```

```typescript
import { parseTranscript, getLastAssistantMessage } from './TranscriptParser';
```

---

### SplitAndTranscribe.ts — Split + Transcribe Large Audio

**Location:** `~/.claude/PAI/Tools/SplitAndTranscribe.ts`

Splits large audio files with FFmpeg then transcribes each chunk via OpenAI Whisper API. Use for files too large for single Whisper call.

```bash
bun ~/.claude/PAI/Tools/SplitAndTranscribe.ts <audio-file>
```

**When to Use:**
- "transcribe this large audio file"
- Files >25MB (Whisper API limit)

---

### IntegrityMaintenance.ts — Integrity Maintenance Background

**Location:** `~/.claude/PAI/Tools/IntegrityMaintenance.ts`

Background script called by `SystemIntegrity.hook.ts` via stdin JSON. Uses AI inference to generate meaningful update documentation for changed files (not templates).

Called automatically by hook — not typically invoked directly.

---

## Utility Libraries & Internal Tools

### pai-fs.ts — JSON File Utilities

**Location:** `~/.claude/PAI/Tools/pai-fs.ts`

Minimal `readJSON` / `writeJSON` helpers with safe error handling. Not a CLI — import as module.

```typescript
import { readJSON, writeJSON } from './pai-fs';
const data = readJSON<MyType>('/path/to/file.json', defaultValue);
writeJSON('/path/to/file.json', data);
```

---

### LoadSkillConfig.ts — Skill Config Loader

**Location:** `~/.claude/PAI/Tools/LoadSkillConfig.ts`

Loads skill JSON/YAML configs merged with user customizations from `SKILLCUSTOMIZATIONS/`. Import as library or use CLI.

```typescript
import { loadSkillConfig } from '~/.claude/PAI/Tools/LoadSkillConfig';
const config = loadSkillConfig<MyConfig>(__dirname, 'config.json');
```

```bash
bun ~/.claude/PAI/Tools/LoadSkillConfig.ts <skill-dir> <filename>
```

---

### SwitchProvider.ts — Researcher Agent Router

**Location:** `~/.claude/PAI/Tools/SwitchProvider.ts`

Reads `PAI/profiles/researchers.yaml` and recommends the best CC agent for a given research task type.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/SwitchProvider.ts                    # list all profiles
bun ~/.claude/PAI/Tools/SwitchProvider.ts technical          # recommend agent
bun ~/.claude/PAI/Tools/SwitchProvider.ts --json comprehensive
```

Task types: `academic`, `technical`, `security`, `current-events`, `market`, `investigative`, `general`, `comprehensive`.

---

### research-catalog.ts — Research Catalog Dispatcher

**Location:** `~/.claude/PAI/Tools/research-catalog.ts`

Finds `MEMORY/CATALOG/` entries with `status: research-queued`, calls PAI Functions API `/v1/research` for each, writes `research_file` path back and sets `status: researched`.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/research-catalog.ts              # process all queued
bun ~/.claude/PAI/Tools/research-catalog.ts --dry-run
bun ~/.claude/PAI/Tools/research-catalog.ts --category algorithm
bun ~/.claude/PAI/Tools/research-catalog.ts --priority high
```

---

### PreviewMarkdown.ts — Markdown Browser Preview

**Location:** `~/.claude/PAI/Tools/PreviewMarkdown.ts`

Converts a markdown file to HTML and opens it in the default browser via a temp file.

```bash
bun ~/.claude/PAI/Tools/PreviewMarkdown.ts /path/to/file.md
```

---

### ExtractTranscript.ts — OpenAI Whisper Transcriber

**Location:** `~/.claude/PAI/Tools/ExtractTranscript.ts`

Transcribes audio/video files via OpenAI Whisper API (cloud). Complements the local `extract-transcript.py` (faster-whisper). Use when API quota is available and accuracy matters more than offline operation.

**Usage:**
```bash
bun ~/.claude/PAI/Tools/ExtractTranscript.ts audio.m4a
bun ~/.claude/PAI/Tools/ExtractTranscript.ts video.mp4 --format srt
bun ~/.claude/PAI/Tools/ExtractTranscript.ts ~/Podcasts/ --batch
```

**Environment Variables:**
- `OPENAI_API_KEY` — required

---

*One-shot agent communication scripts (DelegateSINA.ts, NotifyPNC.ts, SolveTestPoW.ts, TestFeedback.ts) are internal test/delegation scripts for the SINA initiative and PoW system. They write to agent inbox directories directly and are not general-purpose CLIs.*

---

**Last Updated:** 2026-04-24 (120+ tools; 32 new tools added below)

---

## Memory Intelligence & Session Analytics (New — 2026-04-24)

### SessionIndex.ts — FTS5 Full-Text Session Search

**Location:** `~/.claude/PAI/Tools/SessionIndex.ts`

SQLite FTS5 index over all Claude Code JSONL transcripts. Enables verbatim session recall: "what did we decide about X three weeks ago?" — instant keyword search across all past sessions.

**Usage:**
```bash
bun PAI/Tools/SessionIndex.ts --query "Wazuh NFS permission"   # search all sessions
bun PAI/Tools/SessionIndex.ts --rebuild                         # index all transcripts
bun PAI/Tools/SessionIndex.ts --index <path>                    # index one file
bun PAI/Tools/SessionIndex.ts --stats                           # show index stats
bun PAI/Tools/SessionIndex.ts --query "X" --k 20               # return top 20 results
```

**DB:** `~/.claude/MEMORY/STATE/sessions.db` — 8,330+ messages indexed from 118+ files.

---

### InsightsReport.ts — Session Analytics

**Location:** `~/.claude/PAI/Tools/InsightsReport.ts`

Mines `algorithm-reflections.jsonl` for session patterns: effort level distribution, rating trends, criteria pass rates, budget compliance, and recurring failure modes.

**Usage:**
```bash
bun PAI/Tools/InsightsReport.ts              # full report (all sessions)
bun PAI/Tools/InsightsReport.ts --last 30    # last 30 sessions
bun PAI/Tools/InsightsReport.ts --json       # machine-readable output
bun PAI/Tools/InsightsReport.ts --trends     # weekly rating trend chart
```

---

### UserModeler.ts — Honcho-Inspired User Model

**Location:** `~/.claude/PAI/Tools/UserModeler.ts`

Builds a structured user model by mining reflections (effort preferences, topic affinity), memory files (explicit preferences/feedback), and behavioral state files. Writes `MEMORY/STATE/user-model.json`.

**Usage:**
```bash
bun PAI/Tools/UserModeler.ts          # build/refresh model + write JSON
bun PAI/Tools/UserModeler.ts --show   # print existing model
bun PAI/Tools/UserModeler.ts --json   # machine-readable output
```

---

### MemoryQuery.ts — Unified Memory Query

**Location:** `~/.claude/PAI/Tools/MemoryQuery.ts`

Single query interface that fans out to all PAI memory systems: SessionIndex FTS5, auto-memory files, algorithm reflections, and OpenSearch k-NN semantic search. Merges and ranks results by source relevance.

**Usage:**
```bash
bun PAI/Tools/MemoryQuery.ts "Wazuh NFS permission"            # search all sources
bun PAI/Tools/MemoryQuery.ts "docker compose" --sources fts,memory  # specific sources
bun PAI/Tools/MemoryQuery.ts "authentication" --k 5            # limit per source
bun PAI/Tools/MemoryQuery.ts --list-sources                    # check availability
```

**Sources:** `session` (FTS5) | `memory` (auto-memory files) | `reflection` (reflections.jsonl) | `semantic` (SemanticRecall.ts)

---

### SemanticRecall.ts — OpenSearch k-NN Semantic Search

**Location:** `~/.claude/PAI/Tools/SemanticRecall.ts`

k-NN vector search over 1,615+ documents using Gemini Embedding 2 vectors stored in OpenSearch. Returns semantically similar content even without exact keyword matches.

**Usage:**
```bash
bun PAI/Tools/SemanticRecall.ts --query "authentication security" --k 5
bun PAI/Tools/SemanticRecall.ts --query "X" --json
```

**Index:** `pai-memory-vectors` in OpenSearch (127.0.0.1:9200)

---

### GeminiEmbed.ts — Gemini Embedding 2 Vector Generation

**Location:** `~/.claude/PAI/Tools/GeminiEmbed.ts`

CLI wrapper for Gemini Embedding 2 (`text-embedding-004`). Outputs raw embedding vector as JSON. Used by SemanticRecall and BulkIndex pipelines.

**Usage:**
```bash
bun PAI/Tools/GeminiEmbed.ts --text "your text here"   # outputs {"values": [...]}
```

---

### EntityGraph.ts — Local Entity Relationship Graph

**Location:** `~/.claude/PAI/Tools/EntityGraph.ts`

SQLite entity co-occurrence graph built from memory files, reflections, and session transcripts. Maps relationships between PAI-domain entities (tools, services, protocols). 670+ entities, 15,000+ edges.

**Usage:**
```bash
bun PAI/Tools/EntityGraph.ts --rebuild                       # build from scratch
bun PAI/Tools/EntityGraph.ts --query "authentik"             # related entities
bun PAI/Tools/EntityGraph.ts --stats                         # graph statistics
bun PAI/Tools/EntityGraph.ts --path "authentik" "nfs"       # shortest path
bun PAI/Tools/EntityGraph.ts "entity"                        # shorthand for --query
```

**DB:** `~/.claude/MEMORY/STATE/entity-graph.db`

---

### MorningBriefing.ts — Daily Situation Awareness Digest

**Location:** `~/.claude/PAI/Tools/MorningBriefing.ts`

Proactive daily digest: stale open PRDs (>72h idle), last 7-day session stats, low-rated sessions, memory system summary, and quick-action commands. Writes `MEMORY/STATE/briefing-YYYY-MM-DD.md` and sends voice notification.

**Usage:**
```bash
bun PAI/Tools/MorningBriefing.ts          # generate + print today's briefing
bun PAI/Tools/MorningBriefing.ts --show   # print existing briefing
bun PAI/Tools/MorningBriefing.ts --quiet  # write only, no stdout (cron mode)
```

**Cron:** Runs daily at 7am via crontab.

---

### ReflectionSynthesizer.ts — Auto-Draft ISC Improvements

**Location:** `~/.claude/PAI/Tools/ReflectionSynthesizer.ts`

Closes the self-improvement loop: mines reflection patterns ("should have X", "would have X") from `algorithm-reflections.jsonl`, groups recurring failures, and drafts ISC checklist additions as `MEMORY/WORK/ISC-improvements-YYYY-MM-DD.md` for manual review.

**Usage:**
```bash
bun PAI/Tools/ReflectionSynthesizer.ts           # analyze + write draft
bun PAI/Tools/ReflectionSynthesizer.ts --dry-run # print draft, don't write
bun PAI/Tools/ReflectionSynthesizer.ts --min 2   # lower threshold to 2 occurrences
```

**Cron:** Runs weekly Sunday 8am via crontab.

---

## Proactive Intelligence Hooks (New — 2026-04-24)

### LiveContext.hook.ts — Real-Time Session Start Context

**Location:** `~/.claude/hooks/LiveContext.hook.ts`

Fires at SessionStart. Injects compact live system state summary into session context: stale PRDs, user model age warning, today's briefing reference, and memory file count. Registered in `dispatch/SessionStart.ts`.

Output: `{ additionalContext: "..." }` JSON on stdout.

---

## Model Intelligence & Benchmarking (New — 2026-04-19/24)

### ModelMatrix/ModelScout.ts — Local Model Discovery

**Location:** `~/.claude/PAI/Tools/ModelMatrix/ModelScout.ts`

Discovers new/updated Ollama models and scores them against PAI roles (fast inference, reasoning, embedding, etc.). Updates `MEMORY/STATE/model-scout-report.json` and sends voice notification for significant additions.

**Usage:**
```bash
bun PAI/Tools/ModelMatrix/ModelScout.ts           # full scan + notify
bun PAI/Tools/ModelMatrix/ModelScout.ts --quiet   # scan, no notification
bun PAI/Tools/ModelMatrix/ModelScout.ts --json    # JSON output only
```

**Cron:** Runs weekly Sunday 9am via crontab.

---

### ModelBenchmark.ts — AI Model Quality/Latency Benchmark

**Location:** `~/.claude/PAI/Tools/ModelBenchmark.ts`

Benchmarks all available AI models across task categories (reasoning, coding, fast, research). Measures latency, quality score, and tokens/sec (Ollama). Results feed into `MatrixUpdater.ts`.

**Usage:**
```bash
bun PAI/Tools/ModelBenchmark.ts              # full benchmark suite
bun PAI/Tools/ModelBenchmark.ts --category coding   # specific category
bun PAI/Tools/ModelBenchmark.ts --json       # JSON output
```

**Cron:** Runs weekly Sunday 10am via crontab.

---

### MatrixUpdater.ts — Inference Matrix Updater

**Location:** `~/.claude/PAI/Tools/MatrixUpdater.ts`

Ingests benchmark results from `ModelBenchmark.ts` and updates `inference-matrix.json` with top-performing models per category. Feeds into model routing decisions.

```bash
bun PAI/Tools/MatrixUpdater.ts   # reads latest benchmark, updates matrix
```

---

### ParameterTuner.ts — Inference Parameter Optimizer

**Location:** `~/.claude/PAI/Tools/ParameterTuner.ts`

Sweeps over temperature, top_p, and other inference parameters for local Ollama models across task categories to find optimal settings per task type.

```bash
bun PAI/Tools/ParameterTuner.ts --model llama3.2 --category coding
```

---

### TuningPipeline.ts — Full LoRA Tuning Lifecycle

**Location:** `~/.claude/PAI/Tools/TuningPipeline.ts`

Orchestrates full model fine-tuning lifecycle: runs ModelBenchmark, extracts training data via DataDistiller, runs LoRA tuning, and updates matrix with results.

---

## Infrastructure & Monitoring (New — 2026-04-19)

### AssetInventory.ts — Infrastructure Asset Registry

**Location:** `~/.claude/PAI/Tools/AssetInventory.ts`

Manages the PAI infrastructure asset registry. Syncs live state from Tailscale API, HTTP-probes services for health status, and maintains structured inventory.

**Usage:**
```bash
bun PAI/Tools/AssetInventory.ts list     # all assets + services
bun PAI/Tools/AssetInventory.ts seed     # seed known devices from static data
bun PAI/Tools/AssetInventory.ts sync     # sync live state from Tailscale API
bun PAI/Tools/AssetInventory.ts probe    # HTTP-probe services, update status
```

---

### MetricsExporter.ts — PAI Metrics → VictoriaMetrics

**Location:** `~/.claude/PAI/Tools/MetricsExporter.ts`

Reads PAI budget.jsonl, session counts, queue states, and pushes Prometheus-format metrics to VictoriaMetrics remote write endpoint. Enables operational dashboards in Grafana.

**Usage:**
```bash
bun PAI/Tools/MetricsExporter.ts              # push once
bun PAI/Tools/MetricsExporter.ts --dry-run    # print metrics, don't push
```

**Cron:** Runs every 15 minutes via crontab.

---

### GenerateHandoff.ts — Session Handoff Generator

**Location:** `~/.claude/PAI/Tools/GenerateHandoff.ts`

Writes `HANDOFF.md` to each active project's sovereign-data directory and to `~/.claude/MEMORY/HANDOFF.md` for Claude Code pickup. Called by `GenerateHandoff.hook.ts` on every session Stop. Also callable manually.

```bash
bun PAI/Tools/GenerateHandoff.ts   # generate handoffs for all active projects
```

---

### IntegrityMaintenance.ts — System Integrity & Doc Updater

**Location:** `~/.claude/PAI/Tools/IntegrityMaintenance.ts`

Background script that receives session change data via stdin JSON and uses AI inference to generate documentation updates and integrity reports. Called by `DocIntegrity.hook.ts`.

---

### InboxWatchdog.ts — Agent Inbox Monitor

**Location:** `~/.claude/PAI/Tools/InboxWatchdog.ts`

Watches agent inbox directories (`MEMORY/STATE/agent-inbox/{gemini,claude,antigravity,ollama}`) for new messages. Validates PoW, routes to appropriate agent, and logs to AgentLedger.

---

### PipelineOrchestrator.ts — PAI Pipeline Runner

**Location:** `~/.claude/PAI/Tools/PipelineOrchestrator.ts`

Runs multi-step PAI pipelines with real-time monitoring. Supports parallel/sequential step execution, progress tracking, and failure recovery.

---

## Agent Communication (New)

### SwitchboardClient.ts — Inter-Agent WebSocket Communication

**Location:** `~/.claude/PAI/Tools/SwitchboardClient.ts`

Real-time WebSocket client for inter-agent communication via the PAI Switchboard server. Supports channel subscriptions, delegation messages, and broadcast.

**Usage:**
```bash
bun PAI/Tools/SwitchboardClient.ts --channel delegation:gemini --subscribe
bun PAI/Tools/SwitchboardClient.ts --channel council --send "task here"
```

---

## LoRA Fine-Tuning Data Pipeline (New — 2026-04-19)

### DataDistiller.ts — Training Data Extractor

**Location:** `~/.claude/PAI/Tools/DataDistiller.ts`

Scrapes Claude Code project history to extract high-quality input/output pairs for local model fine-tuning (LoRA). Filters by rating, effort level, and task category.

---

### HistoricalDistiller.ts — Historical Session Distiller

**Location:** `~/.claude/PAI/Tools/HistoricalDistiller.ts`

Processes historical JSONL sessions to extract distillation candidates for fine-tuning pipelines. Complements `DataDistiller.ts` for older session data.

---

## Bulk Memory Indexing Tools (New — 2026-04-23)

All `BulkIndex*.ts` tools follow the same pattern: load data from a specific PAI source and batch-upsert into ICM/OpenSearch for semantic recall.

| Tool | Source |
|------|--------|
| `BulkIndexAutoMemory.ts` | Auto-memory MEMORY/*.md files |
| `BulkIndexICM.ts` | ICM memoir entries |
| `BulkIndexKnowledge.ts` | Wisdom frames and domain knowledge |
| `BulkIndexMemoirs.ts` | ICM memoir summaries |
| `BulkIndexPAIMemoryDB.ts` | PAI Memory DB entries |
| `BulkIndexPAIUser.ts` | PAI user profile and preferences |
| `BulkIndexPRDs.ts` | PRD decisions and phase outcomes |
| `BulkIndexReflections.ts` | Algorithm reflection entries |

**Usage pattern:**
```bash
bun PAI/Tools/BulkIndexAutoMemory.ts    # index auto-memory files
bun PAI/Tools/BulkIndexReflections.ts  # index all reflections
```

---

## Special CLI Tools

### algorithm.ts — Algorithm CLI Runner

**Location:** `~/.claude/PAI/Tools/algorithm.ts`

Runs the PAI Algorithm in interactive loop or single-shot mode from the CLI. Enables running Algorithm sessions outside of Claude Code.

```bash
bun PAI/Tools/algorithm.ts --interactive    # interactive REPL mode
bun PAI/Tools/algorithm.ts --task "..."     # single task execution
```

---

## Voice Capture Scripts

### voice-capture.sh — Record Audio → Whisper → Clipboard

**Location:** `~/.claude/PAI/Tools/voice-capture.sh`

Records audio via PulseAudio, transcribes with local Whisper (openai-whisper), and copies transcript to clipboard or writes to voice inbox.

**Prerequisites:** `parecord` (pulseaudio-utils) + `pip install openai-whisper` + `apt install xclip`

**Usage:**
```bash
~/.claude/PAI/Tools/voice-capture.sh                 # record 30s → clipboard
~/.claude/PAI/Tools/voice-capture.sh --duration 60   # record 60s
~/.claude/PAI/Tools/voice-capture.sh --output task   # write to voice inbox
~/.claude/PAI/Tools/voice-capture.sh --check         # check dependencies
```

---

## LocalAssessmentLab - Software Assessment Lab

**Location:** `~/.claude/PAI/Tools/LocalAssessmentLab`

Reusable local capability for software package assessment. It separates host-side static triage, hash/reputation enrichment, isolated VM runtime execution, and host-side returned-evidence ingestion.

**Usage:**
```bash
cd ~/.claude/PAI/Tools/LocalAssessmentLab
./bin/lab-preflight.sh
./bin/lab-new-case.sh xampp-ksis /share/PAI/Shared-Inbox/to-pai/xampp
./bin/lab-static-triage.sh ~/.claude/PAI/MEMORY/WORK/<case-dir>
./bin/lab-static-triage.sh ~/.claude/PAI/MEMORY/WORK/<case-dir> --heavy
./bin/lab-ingest-evidence.sh ~/.claude/PAI/MEMORY/WORK/<case-dir> /path/to/returned/evidence
```

**Current profile:**
- `xampp-ksis`: XAMPP/KSIS assessment workflow using static analysis plus isolated VM runtime evidence.

**Safety boundary:**
- Do not execute submitted software on the host.
- Do not upload proprietary source, secrets, or session state to public services.
- Run dynamic behavior and mutation testing only in an isolated VM with disposable data.
