# Peekaboo - macOS UI Automation

**Location:** ` \openclaw\skills\peekaboo\`  
**Emoji:** 👀  
**Requirements:** `peekaboo` CLI, macOS only  
**Homepage:** https://peekaboo.boo

## Purpose
Full macOS UI automation - capture/inspect screens, target UI elements, drive input, manage apps/windows/menus.

## Installation

```bash
brew install steipete/tap/peekaboo
```

## Prerequisites

Requires permissions:
- **Screen Recording** (System Settings → Privacy & Security)
- **Accessibility** (System Settings → Privacy & Security)

**Check permissions:**
```bash
peekaboo permissions
```

## Quick Start

```bash
# Check permissions
peekaboo permissions

# List apps
peekaboo list apps --json

# Capture & annotate UI
peekaboo see --annotate --path /tmp/peekaboo-see.png

# Click element
peekaboo click --on B1

# Type text
peekaboo type "Hello" --return
```

## Core Commands

### Capture & Vision

| Command | Purpose |
|---------|---------|
| `image` | Screenshot (screen/window/menu bar) |
| `see` | Annotated UI maps with element IDs |
| `capture` | Live capture or video ingest |

### Interaction

| Command | Purpose |
|---------|---------|
| `click` | Click by ID/query/coords |
| `type` | Type text + control keys |
| `press` | Special-key sequences |
| `hotkey` | Modifier combos (cmd+shift+t) |
| `drag` | Drag & drop |
| `swipe` | Gesture-style drags |
| `scroll` | Directional scrolling |
| `move` | Cursor positioning |
| `paste` | Clipboard paste |

### System

| Command | Purpose |
|---------|---------|
| `app` | Launch/quit/hide/unhide/switch |
| `window` | Close/minimize/maximize/move/resize |
| `menu` | Click application menus |
| `menubar` | Click status bar items |
| `dock` | Launch/interact with Dock |
| `dialog` | Interact with system dialogs |
| `space` | List/switch Spaces |
| `clipboard` | Read/write clipboard |
| `open` | Enhanced `open` command |

### Utilities

| Command | Purpose |
|---------|---------|
| `list` | List apps/windows/screens/menubar/permissions |
| `config` | Init/show/edit/validate config |
| `run` | Execute `.peekaboo.json` scripts |
| `tools` | List available tools |
| `bridge` | Inspect Peekaboo Bridge connectivity |
| `clean` | Prune snapshot cache |
| `sleep` | Pause execution |

## Common Targeting Parameters

**App/Window:**
- `--app <name>`
- `--pid <process-id>`
- `--window-title <title>`
- `--window-id <id>`
- `--window-index <index>`

**Snapshot:**
- `--snapshot <id>` (from `see`; defaults to latest)

**Element/Coords:**
- `--on <element-id>` (e.g., B1, T2)
- `--coords x,y`

**Focus:**
- `--no-auto-focus`
- `--space-switch`
- `--bring-to-current-space`

## Reliable Workflow: See → Click → Type

```bash
# 1. Capture & annotate
peekaboo see --app Safari --window-title "Login" --annotate --path /tmp/see.png

# 2. Click element (e.g., B3 = username field)
peekaboo click --on B3 --app Safari

# 3. Type text
peekaboo type "user@example.com" --app Safari

# 4. Tab to next field
peekaboo press tab --count 1 --app Safari

# 5. Type password
peekaboo type "supersecret" --app Safari --return
```

## Capture & Analysis

### Screenshot

```bash
# Full screen
peekaboo image --mode screen --screen-index 0 --retina --path /tmp/screen.png

# Specific window
peekaboo image --app Safari --window-title "Dashboard" --path /tmp/window.png

# With AI analysis
peekaboo image --app Safari --window-title "Dashboard" --analyze "Summarize KPIs"
```

### Annotated UI Map

```bash
peekaboo see --mode screen --screen-index 0 --annotate --path /tmp/ui-map.png
```

### Live Capture (Motion-Aware)

```bash
peekaboo capture live --mode region --region 100,100,800,600 --duration 30 \
  --active-fps 8 --idle-fps 2 --highlight-changes --path /tmp/capture
```

## App & Window Management

```bash
# Launch app
peekaboo app launch "Safari" --open https://example.com

# Focus window
peekaboo window focus --app Safari --window-title "Example"

# Resize window
peekaboo window set-bounds --app Safari --x 50 --y 50 --width 1200 --height 800

# Quit app
peekaboo app quit --app Safari
```

## Menus & Dock

```bash
# Click menu item
peekaboo menu click --app Safari --item "New Window"

# Navigate menu path
peekaboo menu click --app TextEdit --path "Format > Font > Show Fonts"

# Click menu extra (status bar)
peekaboo menu click-extra --title "WiFi"

# Launch from Dock
peekaboo dock launch Safari

# List menubar items
peekaboo menubar list --json
```

## Input Automation

### Mouse

```bash
# Move cursor (smooth)
peekaboo move 500,300 --smooth

# Drag between elements
peekaboo drag --from B1 --to T2

# Swipe gesture
peekaboo swipe --from-coords 100,500 --to-coords 100,200 --duration 800

# Scroll
peekaboo scroll --direction down --amount 6 --smooth
```

### Keyboard

```bash
# Hotkey combo
peekaboo hotkey --keys "cmd,shift,t"

# Special key
peekaboo press escape

# Type with delay
peekaboo type "Line 1\nLine 2" --delay 10

# Clear before typing
peekaboo type "New text" --clear
```

## Global Runtime Flags

- `--json` / `-j` - JSON output
- `--verbose` / `-v` - Verbose logging
- `--log-level <level>` - Set log level
- `--no-remote` - Disable remote bridge
- `--bridge-socket <path>` - Custom bridge socket

## Best Practices

1. **Use `see --annotate`** to identify targets before clicking
2. **Target by window** for reliable automation
3. **Add delays** for UI transitions
4. **Check permissions** first
5. **JSON output** for scripting

## Source
SKILL.md: ` \openclaw\skills\peekaboo\SKILL.md`
