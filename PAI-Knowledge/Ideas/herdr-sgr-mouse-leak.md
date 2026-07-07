---
title: "Herdr v0.6.0 SGR mouse escape sequence leak"
type: idea
tags: [herdr, terminal, sgr, mouse, multiplexer, escape-sequences]
created: 2026-05-25
updated: 2026-05-25
quality: 5
source_session: 20260525-125900_antigravity-cli-integration-gaps
related:
  - slug: herdr-mouse-capture-config
    type: relates
  - slug: terminal-multiplexer-escape-handling
    type: relates
---

# Herdr v0.6.0 SGR mouse escape sequence leak

## Thesis
Herdr v0.6.0 enables SGR extended mouse mode (DECSET 1006) on the outer terminal, but under certain conditions the raw SGR escape sequences (`<5;X;YM`) leak through as visible text in panes instead of being consumed by Herdr's TUI event handler. The leak occurs because escape sequences sent from inside a pane are consumed by Herdr's virtual terminal emulation layer and never reach the outer terminal, so `printf '\x1b[?1000l\x1b[?1006l'` inside a pane cannot disable SGR at the terminal level.

## Evidence
- Observed patterns: `5;52;14M`, `35;109;23M`, `1;22M`, `8M`, `[<35;109;24M` — all SGR extended mode (DECSET 1006) mouse events where `ESC[` prefix is consumed but parameters render as text
- `5` = scroll wheel down (standard xterm button code)
- Coordinates span multiple positions across the TUI (pane areas + sidebar)
- Occurs in ALL panes, not specific to any one app
- Setting `mouse_capture = false` in Herdr config breaks pane selection but does NOT stop the leak (config reload doesn't re-initialize terminal SGR mode)
- Setting `tmux set -g mouse off` has no effect (tmux runs INSIDE Herdr panes for OmniPulse, not as the host)
- `printf '\x1b[?1000l\x1b[?1006l'` from inside a Herdr pane is consumed by Herdr's pty layer — never reaches outer terminal

## Root cause analysis
The outer terminal emulator (SSH client side) has SGR mouse mode enabled by Herdr's startup initialization. When mouse events arrive:
1. Herdr normally consumes them for its TUI
2. Under certain conditions (race, buffer overflow, mode conflict), the raw sequences bypass Herdr's handler
3. The `ESC[` prefix is consumed but the parameter body (`5;X;YM`) renders as visible text

The `mouse_capture` setting in Herdr's config controls whether Herdr REACTS to mouse events for pane selection/resizing, but does NOT control whether SGR mode is ENABLED on the outer terminal.

## Tried fixes (none worked)
- `mouse_capture = false` in config.toml — breaks pane selection, leak continues
- `tmux set -g mouse off` — irrelevant (tmux is inside Herdr's panes, not hosting Herdr)  
- `printf '\x1b[?1000l\x1b[?1006l'` in a pane — consumed by Herdr's pty, never reaches terminal
- `herdr kill` — invalid command (correct command: `herdr server stop`)

## Untried potential fixes
1. **Proper server stop** (`herdr server stop`) — should trigger terminal cleanup (SGR disable) on server shutdown, then restart clean
2. **`TERM=vt100 herdr`** — force terminal type without SGR mouse support to prevent Herdr from enabling SGR mode
3. **`--no-session` flag** — run monolithically, might bypass SGR initialization path

## Implications
- Herdr's terminal initialization sets SGR mode but its cleanup path when config-reloading doesn't disable it
- The `mouse_capture` config option name is misleading — it controls mouse UI interaction, not SGR mode enabling
- This is likely a Herdr v0.6.0 bug worth reporting (SGR mode should be toggled based on `mouse_capture` setting at startup, or a separate `enable_sgr_mouse` config option should exist)
