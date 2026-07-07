# Voice STT (Speech-to-Text) in OpenCode Web

Talk to PAI using your voice! The OpenCode web interface now includes speech-to-text capability.

## Features

✅ **Browser-based STT** using Web Speech API  
✅ **Real-time transcription** with interim results  
✅ **Auto-send** after 2 seconds of silence  
✅ **Visual feedback** during recording  
✅ **Keyboard shortcuts** for quick access  
✅ **Fallback to clipboard** if chat input not found  

## How to Use

### Method 1: Click Button

1. **Open OpenCode web**: https://pai-primary.taild73141.ts.net/
2. **Click microphone button** (bottom-right, blue circle with 🎤)
3. **Speak your message**
4. **Wait 2 seconds** of silence → Auto-stops
5. **Click "Send"** → Message sent to chat

### Method 2: Keyboard Shortcut

1. Press **Ctrl+Shift+M** (or **Cmd+Shift+M** on Mac)
2. Speak your message
3. Press **Ctrl+Shift+M** again to stop
4. Click "Send"

### Method 3: Quick Send

1. Click microphone
2. Speak
3. Wait for auto-stop
4. Press **Enter** to send immediately

## User Interface

### Microphone Button (Bottom-Right)

**States:**

**Idle:**
```
🎤 (blue circle)
```

**Recording:**
```
⏺️ (red circle, pulsing)
```

### Transcript Display (Above Button)

Shows while recording:
```
┌─────────────────────────────────┐
│ 🎤 Listening...                 │
│                                  │
│ This is what you're saying...   │
│ (interim text in gray)          │
│                                  │
│ [  Send  ]  [ Cancel ]          │
└─────────────────────────────────┘
```

## Behavior

### Auto-Stop After Silence

After **2 seconds of silence**, recording automatically stops:
- Status changes to: ✓ Ready to send
- Transcript remains visible
- Click "Send" or "Cancel"

### Continuous Listening

While you're speaking, it keeps listening:
- No timeout while audio detected
- Supports long messages
- Pauses are preserved

### Interim vs Final Results

**Final (white text):**
- Confirmed transcription
- Won't change

**Interim (gray text):**
- Still processing
- May change as you continue speaking

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| **Ctrl+Shift+M** | Toggle microphone on/off |
| **Escape** | Cancel and close |
| **Enter** | Send message (when stopped) |

## Browser Compatibility

**Supported:**
- ✅ Chrome/Edge (Chromium)
- ✅ Safari 14.1+
- ⚠️ Firefox (limited support)

**Not supported:**
- ❌ Firefox < 123
- ❌ Older browsers

**Check support:**
```javascript
if (window.SpeechRecognition || window.webkitSpeechRecognition) {
  console.log('STT supported!');
}
```

## Privacy & Permissions

### Microphone Permission

Browser will request microphone access on first use:
```
┌───────────────────────────────────┐
│ Allow pai-primary.taild73141.ts.net │
│ to use your microphone?           │
│                                    │
│  [ Block ]  [ Allow ]             │
└───────────────────────────────────┘
```

Click **Allow** to enable voice input.

### Where Audio Goes

**Browser-based STT:**
- Audio sent to browser's speech API
- Usually Google Cloud Speech (Chrome) or Apple Dictation (Safari)
- **NOT** processed locally
- Subject to browser vendor's privacy policy

**Future:** Local Whisper for privacy-preserving STT

## Configuration

### Language

Currently hardcoded to English (US):
```javascript
recognition.lang = 'en-US';
```

**To change:**
Edit `~/.config/opencode/plugins/pai-voice-stt.js`:
```javascript
recognition.lang = 'en-GB';  // British English
recognition.lang = 'es-ES';  // Spanish
recognition.lang = 'fr-FR';  // French
```

### Silence Timeout

Default: 2 seconds

**To change:**
Edit plugin:
```javascript
silenceTimeout = setTimeout(() => {
  // ...
}, 3000);  // 3 seconds
```

## Integration with Chat

The plugin tries multiple methods to send text to chat:

### Method 1: Find Textarea
```javascript
textarea[placeholder*="message"]
textarea[placeholder*="chat"]
textarea[data-testid="chat-input"]
#chat-input
.chat-input
```

### Method 2: Simulate Enter Key
If textarea found but no send button:
```javascript
KeyboardEvent('Enter')
```

### Method 3: Clipboard Fallback
If chat input not found:
```javascript
navigator.clipboard.writeText(transcript)
// Shows: 📋 Copied to clipboard
```

## Troubleshooting

**Microphone button not appearing:**
```javascript
// Check browser console
[PAI STT] Browser does not support Web Speech API
```
→ Use Chrome/Edge/Safari

**"No speech detected" error:**
- Check microphone is connected
- Check browser permissions
- Try speaking louder
- Check system audio input

**Transcript not sending to chat:**
```javascript
[PAI STT] Chat input not found, copying to clipboard
```
→ Paste manually (Ctrl+V)

**Recording won't stop:**
- Click button again
- Press Escape
- Refresh page

**Audio cutting out:**
- Continuous mode restarts on silence
- Speak more clearly
- Reduce background noise

## Advanced Usage

### Voice Commands (Future)

Planned support for commands:
- "Send message to PNC"
- "Search for X"
- "Open session Y"
- "Run command Z"

### Whisper API Fallback (Future)

For browsers without Web Speech API:
```javascript
// Record audio
// Send to OpenAI Whisper API
// Return transcription
```

## Files

| File | Purpose |
|------|---------|
| `~/.config/opencode/plugins/pai-voice-stt.js` | STT plugin |
| `~/.config/opencode/opencode.json` | Plugin config |

## Plugin Code Structure

```javascript
export default {
  name: 'pai-voice-stt',
  
  async onLoad() {
    // Initialize SpeechRecognition
    // Create UI elements (button, transcript display)
    // Set up event handlers
    // Add keyboard shortcuts
  }
};
```

## Comparison: TTS vs STT

| Feature | TTS (Text-to-Speech) | STT (Speech-to-Text) |
|---------|---------------------|---------------------|
| **Plugin** | `pai-voice-embed.js` | `pai-voice-stt.js` |
| **Direction** | PAI → You (output) | You → PAI (input) |
| **UI** | Widget + notifications | Button + transcript |
| **Shortcut** | N/A | Ctrl+Shift+M |
| **Backend** | Legion Voicebox | Browser Speech API |
| **Fallback** | Browser speechSynthesis | Clipboard copy |

## Complete Voice System

**Output (TTS):**
- PAI speaks to you
- Audio plays in browser
- Visual notifications

**Input (STT):**
- You speak to PAI
- Voice → text transcription
- Auto-send to chat

**Bi-directional voice conversation now possible!**

## Example Workflow

1. **PAI sends voice message** →  
   🔊 Widget flashes, audio plays, notification shows

2. **You respond by voice** →  
   Click 🎤, speak, auto-send

3. **PAI processes and responds** →  
   Text response + optional voice

4. **Continue conversation** →  
   Mix text and voice as needed

## Future Enhancements

- [ ] Local Whisper for privacy
- [ ] Voice command detection
- [ ] Multi-language support selector
- [ ] Voice activity detection (auto-start on speech)
- [ ] Noise cancellation
- [ ] Custom wake word ("Hey PAI...")
- [ ] Voice profiles (recognize who's speaking)
- [ ] Integration with voice router (route input by speaker)
