# Gifgrep - GIF Search & Download

**Location:** ` \openclaw\skills\gifgrep\`  
**Emoji:** 🧲  
**Requirements:** `gifgrep` CLI  
**Homepage:** https://gifgrep.com

## Purpose
Search GIF providers (Tenor/Giphy), browse TUI, download results, and extract stills/sheets.

## Installation

```bash
brew install steipete/tap/gifgrep
```

**Or via Go:**
```bash
go install github.com/steipete/gifgrep/cmd/gifgrep@latest
```

## GIF-Grab Workflow

Search → preview → download → extract (still/sheet) for fast review and sharing.

## Quick Start

**Basic search:**
```bash
gifgrep cats --max 5
```

**URL output:**
```bash
gifgrep cats --format url | head -n 5
```

**JSON output:**
```bash
gifgrep search --json cats | jq '.[0].url'
```

**TUI (Interactive):**
```bash
gifgrep tui "office handshake"
```

**Download:**
```bash
gifgrep cats --download --max 1 --format url
```

## TUI & Previews

**TUI mode:**
```bash
gifgrep tui "query"
```

**CLI still previews** (Kitty/Ghostty only):
```bash
gifgrep cats --thumbs
```

## Download & Reveal

```bash
# Download to ~/Downloads
gifgrep funny --download --max 5

# Download & show in Finder
gifgrep funny --download --reveal --max 1
```

## Stills & Sheets

### Extract Single Frame

```bash
gifgrep still ./clip.gif --at 1.5s -o still.png
```

### Create Sheet (Grid of Frames)

```bash
gifgrep sheet ./clip.gif --frames 9 --cols 3 -o sheet.png
```

**Sheets** = Single PNG grid of sampled frames (great for quick review, docs, PRs, chat).

**Tuning:**
- `--frames <count>` - Number of frames to extract
- `--cols <width>` - Grid width
- `--padding <spacing>` - Spacing between frames

## Providers

**Auto (default):**
```bash
gifgrep cats
```

**Specific provider:**
```bash
gifgrep cats --source tenor
gifgrep cats --source giphy
```

**API Keys:**
- `GIPHY_API_KEY` - Required for `--source giphy`
- `TENOR_API_KEY` - Optional (demo key used if unset)

## Output Formats

**JSON:**
```bash
gifgrep cats --json
```

Returns array with:
- `id`, `title`, `url`, `preview_url`, `tags`, `width`, `height`

**Pipe-friendly:**
```bash
gifgrep cats --format url
```

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `GIFGREP_SOFTWARE_ANIM=1` | Force software animation |
| `GIFGREP_CELL_ASPECT=0.5` | Tweak preview geometry |

## Best Practices

1. **TUI for browsing** - Interactive selection
2. **CLI for scripting** - JSON output + --format url
3. **Sheets for review** - Quick visual overview
4. **Stills for thumbnails** - Extract specific frames
5. **Download + reveal** - Quick access to files

## Common Use Cases

**Find reaction GIF:**
```bash
gifgrep tui "shocked reaction"
```

**Download top result:**
```bash
gifgrep "happy dance" --download --max 1 --reveal
```

**Create preview sheet:**
```bash
gifgrep "cat fail" --download --max 1
gifgrep sheet ~/Downloads/cat-fail.gif --frames 12 --cols 4 -o preview.png
```

**Extract thumbnail:**
```bash
gifgrep still ~/Downloads/clip.gif --at 0.5s -o thumb.png
```

## Source
SKILL.md: ` \openclaw\skills\gifgrep\SKILL.md`
