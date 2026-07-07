# Voice Embedded in OpenCode Web GUI

Voice playback is now built directly into the OpenCode web interface - no separate tab needed!

## What It Does

When you use OpenCode in your browser, voice messages automatically:
- ✅ Play audio inline
- ✅ Show visual notifications
- ✅ Provide volume control
- ✅ Work with browser's built-in TTS as fallback

## How It Works

The `pai-voice-embed.js` plugin loads automatically when OpenCode web starts:

1. **Auto-connects** to voice router on page load
2. **Registers** browser as audio-capable client
3. **Polls** for voice messages every 2 seconds
4. **Plays** audio when messages arrive
5. **Shows** visual notification banners

## User Interface

### Voice Widget (Bottom Right)

A small widget appears in the bottom-right corner:

```
🔊 Voice Ready  [━━━━━━━━] 80%
```

**States:**
- 🔇 Voice Disconnected (gray border)
- 🔊 Voice Ready (green border)

**Click** the widget to show/hide volume slider.

### Notification Banners (Top Right)

When a voice message arrives:
```
┌─────────────────────────────────┐
│ 🔊 PAI Voice               [×] │
│ Your message appears here       │
└─────────────────────────────────┘
```

- Auto-dismiss after 10 seconds
- Click [×] to dismiss manually
- Stacks multiple notifications

## Configuration

### Plugin Location

```
~/.config/opencode/plugins/pai-voice-embed.js
```

### Enable/Disable

Edit `~/.config/opencode/opencode.json`:

```json
{
  "plugin": [
    "opencode-claude-auth@1.5.4",
    "./plugins/pai-voice-embed.js"  // ← Remove this line to disable
  ]
}
```

Restart OpenCode web:
```bash
systemctl --user restart opencode-web.service
```

### Volume Persistence

Volume setting is saved in browser localStorage:
```javascript
localStorage.getItem('pai-voice-volume')  // 0.0 to 1.0
```

## Audio Playback

### Priority 1: Server-Generated Audio

If Legion Voicebox provides an audio URL:
```javascript
audio.src = message.audio_url;
audio.play();
```

### Priority 2: Browser TTS Fallback

If no audio URL, uses browser's built-in speech synthesis:
```javascript
const utterance = new SpeechSynthesisUtterance(message.text);
window.speechSynthesis.speak(utterance);
```

## Client Registration

The plugin registers with the voice router as an audio-capable browser client:

```json
{
  "client_id": "opencode_1783293519627_abc123",
  "type": "browser",
  "capabilities": {
    "audio": true,
    "display": true
  }
}
```

### Heartbeat

Sends heartbeat every 30 seconds to maintain connection:
```
POST http://localhost:31337/voice/heartbeat
{"client_id": "opencode_..."}
```

### Polling

Polls for messages every 2 seconds:
```
GET http://localhost:31337/voice/poll?client_id=opencode_...
```

## Integration with Voice Router

The voice router (future implementation) will:
1. Detect active browser clients
2. Route voice messages to browser instead of Legion speakers
3. Deliver audio URLs via polling endpoint

## Browser Autoplay Policy

Modern browsers block autoplay audio. The plugin handles this:

**If autoplay works:**
- Audio plays immediately

**If autoplay blocked:**
- Visual notification still appears
- Falls back to browser TTS
- User can click to enable audio

## Multiple Tabs

If you have multiple OpenCode tabs open:
- Each tab registers as a separate client
- All tabs receive voice messages
- Audio plays in all tabs (might want to fix this in future)

**Future enhancement:** Only play in the active/focused tab.

## Debugging

### Check Plugin Loaded

Open browser console:
```
[PAI Voice] Initializing voice player...
[PAI Voice] Connected to voice router
[PAI Voice] Voice player embedded successfully
```

### Check Registration

```bash
# On server
curl http://localhost:31337/voice/clients
# Should show registered opencode_* clients
```

### Test Voice

```bash
curl -X POST http://localhost:31337/notify \
  -H "Content-Type: application/json" \
  -d '{"message":"Test from embedded player","voice_enabled":true}'
```

**Expected:**
1. Notification appears in browser
2. Audio plays (or TTS speaks)
3. Widget flashes briefly

### Check Console

Browser DevTools → Console:
```
[PAI Voice] Playing message: Test from embedded player
```

## Files Modified

| File | Change |
|------|--------|
| `~/.config/opencode/plugins/pai-voice-embed.js` | New plugin |
| `~/.config/opencode/opencode.json` | Added to `plugin` array |
| `~/.config/systemd/user/opencode-web.service` | Restart to load plugin |

## API Endpoints Used

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/voice/register` | POST | Register browser client |
| `/voice/heartbeat` | POST | Keep connection alive |
| `/voice/poll` | GET | Fetch pending messages |
| `/notify` | POST | Send voice message (testing) |

## Comparison: Standalone vs Embedded

**Standalone Player** (`/voice/player`):
- Separate browser tab
- Full-featured interface
- Message history
- Manual connection

**Embedded Player** (this):
- Built into OpenCode
- Lightweight widget
- Auto-connects
- No extra tab needed

## Future Enhancements

- [ ] WebSocket/SSE instead of polling
- [ ] Play only in active tab
- [ ] Mute/unmute button
- [ ] Voice message queue
- [ ] Replay last message
- [ ] Notification positioning preferences
- [ ] Dark/light theme support
- [ ] Custom notification sounds
