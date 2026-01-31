# Camsnap - RTSP/ONVIF Camera Capture

**Location:** ` \openclaw\skills\camsnap\`  
**Emoji:** 📸  
**Requirements:** `camsnap` CLI, `ffmpeg`  
**Homepage:** https://camsnap.ai

## Purpose
Capture frames, clips, or motion events from RTSP/ONVIF cameras.

## Installation

```bash
brew install steipete/tap/camsnap
```

## Setup

**Config file:** `~/.config/camsnap/config.yaml`

**Add camera:**
```bash
camsnap add --name kitchen --host 192.168.0.10 --user user --pass pass
```

## Common Commands

### Discover Cameras

```bash
camsnap discover --info
```

### Snapshot

```bash
camsnap snap kitchen --out shot.jpg
```

### Video Clip

```bash
camsnap clip kitchen --dur 5s --out clip.mp4
```

### Motion Watch

```bash
camsnap watch kitchen --threshold 0.2 --action '...'
```

### Doctor (Diagnostics)

```bash
camsnap doctor --probe
```

## Best Practices

1. **Requires ffmpeg** - Install via `brew install ffmpeg`
2. **Test with short capture** before longer clips
3. **Configure cameras** in `~/.config/camsnap/config.yaml`
4. **Use discover** to find cameras on network
5. **Motion threshold** 0.0-1.0 (0.2 is sensitive, 0.5 is moderate)

## Common Use Cases

**Security snapshot:**
```bash
camsnap snap front-door --out /tmp/front-$(date +%Y%m%d-%H%M%S).jpg
```

**5-second clip:**
```bash
camsnap clip garage --dur 5s --out /tmp/garage-clip.mp4
```

**Motion detection:**
```bash
camsnap watch backyard --threshold 0.3 --action 'notify "Motion detected in backyard"'
```

## Troubleshooting

**Camera not found:**
- Use `camsnap discover` to find cameras
- Verify network connectivity
- Check camera credentials

**ffmpeg not found:**
```bash
brew install ffmpeg
```

**Connection timeout:**
- Check camera IP/hostname
- Verify credentials
- Ensure camera is RTSP/ONVIF compatible

## Source
SKILL.md: ` \openclaw\skills\camsnap\SKILL.md`
