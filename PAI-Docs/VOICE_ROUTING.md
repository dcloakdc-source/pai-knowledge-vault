# PAI Voice Routing

Smart audio playback routing based on your current location and context.

## The Problem

Voice TTS is generated on Legion (100.99.240.58:17493) but you're not always physically at Legion. Audio needs to reach you wherever you are:
- SSH'd into pai-primary from Office
- Working in OpenCode web GUI
- On mobile/tablet
- At Legion desktop

## Solution: Multi-Route Voice System

### Route 1: SSH Terminal Notification (Current Default)

When SSH'd in, voice messages appear as terminal notifications instead of playing audio you can't hear.

**Test:**
```bash
~/.claude/PAI/Tools/voice-notify-ssh.sh "Hello from PAI"
```

**Output:**
```
🔔 PAI Voice: Hello from PAI
```

**Force this mode:**
```bash
curl -X POST http://localhost:31337/notify \
  -H "Content-Type: application/json" \
  -d '{"message":"Test","voice_enabled":true,"route":"ssh"}'
```

### Route 2: Browser Audio Player

Open the voice player in your browser to receive audio:

**URL:**
```
https://pai-primary.taild73141.ts.net/voice/player
http://localhost:31337/voice/player
```

**Features:**
- Live audio playback
- Volume control
- Message history
- Auto-play when connected

**How it works:**
1. Open voice player in browser
2. Player registers with voice router
3. Future voice messages route to browser
4. Audio plays automatically

### Route 3: Cast to Network Device

Cast TTS audio to Office Display or other cast-enabled device.

**Configuration:**
Edit `/home/duane/PAI/PULSE/PULSE.toml`:
```toml
[voice]
cast_enabled = true
cast_device = "192.168.50.233"  # Office Display IP
```

**Force this mode:**
```bash
curl -X POST http://localhost:31337/notify \
  -H "Content-Type: application/json" \
  -d '{"message":"Test","voice_enabled":true,"cast":true}'
```

### Route 4: Local Legion Playback (Fallback)

When you're physically at Legion, audio plays on Legion speakers.

**Force this mode:**
```bash
curl -X POST http://localhost:31337/notify \
  -H "Content-Type: application/json" \
  -d '{"message":"Test","voice_enabled":true,"force_audio":true}'
```

## Automatic Routing

The voice system detects your location and routes intelligently:

1. **Browser active** → Play in browser
2. **SSH session** → Terminal notification
3. **Cast device available** → Cast to device
4. **Fallback** → Legion speakers

## Manual Override

Force a specific route by adding `route` parameter:

```json
{
  "message": "Your message",
  "voice_enabled": true,
  "route": "browser" | "ssh" | "cast" | "local" | "none"
}
```

## Configuration

### Enable/Disable Voice

**Globally disable:**
Edit `/home/duane/PAI/PULSE/PULSE.toml`:
```toml
[voice]
enabled = false
```

**Per-message disable:**
```json
{"message": "...", "voice_enabled": false}
```

### Set Default Voice

```toml
[voice]
default_voice_id = "algorithm"  # or "main", "ollama", "antigravity"
```

### Voice Profiles

Available voices (configured in Legion Voicebox):
- `algorithm` - af_sky (default)
- `main` - Same as algorithm
- `ollama` - am_michael
- `antigravity` - am_adam

## Usage Patterns

### Pattern 1: SSH Work Session

You're SSH'd into pai-primary from Office laptop:

```bash
# Voice messages appear as terminal notifications
# No configuration needed - auto-detected
```

### Pattern 2: Web GUI Session

You're using OpenCode web interface:

```bash
# Open voice player
firefox https://pai-primary.taild73141.ts.net/voice/player

# Voice messages play in browser automatically
```

### Pattern 3: Office Work (Cast)

You're at Office desk with Display available:

```bash
# Enable cast in config
# Voice messages cast to Office Display
```

### Pattern 4: Mobile Notification

You're away from desk:

```bash
# Future: Telegram/ntfy notification with audio link
# Not yet implemented
```

## Files

| File | Purpose |
|------|---------|
| `/home/duane/PAI/PULSE/VoiceServer/voice.ts` | Voice routing logic |
| `/home/duane/PAI/PULSE/modules/voice-router.ts` | Smart router (future) |
| `~/.claude/PAI/Tools/voice-notify-ssh.sh` | SSH terminal notifications |
| `~/.config/opencode/web-extensions/voice-player.html` | Browser audio player |
| `/home/duane/PAI/PULSE/PULSE.toml` | Voice configuration |

## Troubleshooting

**Not hearing audio:**
```bash
# Check which route is active
curl http://localhost:31337/voice/route

# Force browser route
# Open: https://pai-primary.taild73141.ts.net/voice/player
```

**Terminal notifications not showing:**
```bash
# Test the script directly
~/.claude/PAI/Tools/voice-notify-ssh.sh "Test"

# Check if terminal bell works
printf '\a'
```

**Browser player not connecting:**
```bash
# Check Pulse is running
systemctl --user status pai-pulse.service

# Check the endpoint
curl http://localhost:31337/voice/player
```

## Future Enhancements

- [ ] WebSocket/SSE for real-time browser delivery
- [ ] Mobile app integration (Telegram/ntfy)
- [ ] Multi-device simultaneous playback
- [ ] Voice message queue/replay
- [ ] Spatial audio routing
- [ ] Voice activity detection (speak only when active)
