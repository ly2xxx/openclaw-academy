# Sonos CLI - Smart Speaker Control

**Location:** ` \openclaw\skills\sonoscli\`  
**Emoji:** 🔊  
**Requirements:** `sonos` CLI  
**Homepage:** https://sonoscli.sh

## Purpose
Control Sonos speakers on local network - discover, play, volume, grouping.

## Installation

```bash
go install github.com/steipete/sonoscli/cmd/sonos@latest
```

## Quick Start

**Discover speakers:**
```bash
sonos discover
```

**Check status:**
```bash
sonos status --name "Kitchen"
```

**Playback control:**
```bash
sonos play --name "Kitchen"
sonos pause --name "Kitchen"
sonos stop --name "Kitchen"
```

**Volume control:**
```bash
sonos volume set 15 --name "Kitchen"
```

## Common Tasks

### Grouping

```bash
# Group status
sonos group status

# Join group
sonos group join --name "Bedroom" --to "Kitchen"

# Leave group
sonos group unjoin --name "Bedroom"

# Party mode (all speakers)
sonos group party

# Solo mode (ungroup all)
sonos group solo
```

### Favorites

```bash
# List favorites
sonos favorites list

# Play favorite
sonos favorites open --name "Morning Playlist"
```

### Queue Management

```bash
# List queue
sonos queue list --name "Kitchen"

# Play queue
sonos queue play --name "Kitchen"

# Clear queue
sonos queue clear --name "Kitchen"
```

### Spotify Search (SMAPI)

```bash
sonos smapi search --service "Spotify" --category tracks "query"
```

**Optional:** Spotify Web API search requires `SPOTIFY_CLIENT_ID` and `SPOTIFY_CLIENT_SECRET`.

## Troubleshooting

**If SSDP discovery fails:**
```bash
sonos status --ip 192.168.1.100
```

Specify speaker IP directly.

## Best Practices

1. **Use --name** for speaker targeting
2. **Discover first** to get speaker names
3. **Volume levels** 0-100 (15-30 typical for background)
4. **Group operations** for multi-room audio
5. **Queue management** for playlist control

## Common Use Cases

**Morning routine:**
```bash
sonos play --name "Kitchen"
sonos volume set 20 --name "Kitchen"
sonos favorites open --name "Morning Jazz"
```

**Party mode:**
```bash
sonos group party
sonos volume set 40 --name "Living Room"
sonos play --name "Living Room"
```

**Quiet time:**
```bash
sonos volume set 10 --name "Bedroom"
sonos pause --name "Living Room"
```

## Source
SKILL.md: ` \openclaw\skills\sonoscli\SKILL.md`
