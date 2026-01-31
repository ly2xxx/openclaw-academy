# Spotify Player CLI

**Location:** ` \openclaw\skills\spotify-player\`  
**Emoji:** 🎵  
**Requirements:** `spogo` (preferred) or `spotify_player`, Spotify Premium  
**Homepage:** https://www.spotify.com

## Purpose
Terminal Spotify playback and search control.

## Installation

**Preferred (spogo):**
```bash
brew tap steipete/tap
brew install spogo
```

**Fallback (spotify_player):**
```bash
brew install spotify_player
```

## Setup

**spogo authentication:**
```bash
spogo auth import --browser chrome
```

## spogo Commands (Preferred)

### Search

```bash
spogo search track "query"
```

### Playback

```bash
spogo play
spogo pause
spogo next
spogo prev
```

### Devices

```bash
# List devices
spogo device list

# Set device
spogo device set "<name|id>"
```

### Status

```bash
spogo status
```

## spotify_player Commands (Fallback)

### Search

```bash
spotify_player search "query"
```

### Playback

```bash
spotify_player playback play
spotify_player playback pause
spotify_player playback next
spotify_player playback previous
```

### Connect Device

```bash
spotify_player connect
```

### Like Track

```bash
spotify_player like
```

## Configuration

**Config folder:** `~/.config/spotify-player/`  
**Main config:** `app.toml`

**For Spotify Connect integration:**
Set user `client_id` in config.

**TUI shortcuts:** Press `?` in app for help.

## Requirements

1. **Spotify Premium account** required
2. Either `spogo` or `spotify_player` installed
3. Valid Spotify credentials

## Best Practices

1. **Use spogo** (lighter, faster)
2. **Import browser cookies** for easy auth
3. **Check device list** before playing
4. **Set default device** for quick playback

## Source
SKILL.md: ` \openclaw\skills\spotify-player\SKILL.md`
