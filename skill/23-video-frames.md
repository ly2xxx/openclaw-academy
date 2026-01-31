# Video Frames - Extract Frames from Videos

**Location:** ` \openclaw\skills\video-frames\`  
**Emoji:** 🎞️  
**Requirements:** `ffmpeg` CLI  
**Homepage:** https://ffmpeg.org

## Purpose
Extract frames or short clips from videos using ffmpeg.

## Installation

```bash
brew install ffmpeg
```

## Quick Start

**First frame:**
```bash
{baseDir}/scripts/frame.sh /path/to/video.mp4 --out /tmp/frame.jpg
```

**Frame at timestamp:**
```bash
{baseDir}/scripts/frame.sh /path/to/video.mp4 --time 00:00:10 --out /tmp/frame-10s.jpg
```

## Usage Patterns

### Timestamps

**Format:** `HH:MM:SS` or `HH:MM:SS.mmm`

**Examples:**
- `--time 00:00:10` - 10 seconds
- `--time 00:01:30` - 1 minute 30 seconds
- `--time 00:00:05.500` - 5.5 seconds

### Output Formats

**JPEG (quick share):**
```bash
--out /tmp/frame.jpg
```

**PNG (crisp UI frames):**
```bash
--out /tmp/frame.png
```

## Notes

1. **Use --time** for "what's happening at this moment?"
2. **JPEG** for quick sharing (smaller file size)
3. **PNG** for crisp UI screenshots (lossless)

## Best Practices

1. **Prefer --time** over first frame for analysis
2. **Use JPEG** for photos/natural content
3. **Use PNG** for screenshots/UI elements
4. **Check video length** before extracting (to avoid errors)

## Common Use Cases

**Thumbnail generation:**
```bash
{baseDir}/scripts/frame.sh video.mp4 --time 00:00:01 --out thumbnail.jpg
```

**UI inspection:**
```bash
{baseDir}/scripts/frame.sh screencast.mp4 --time 00:02:30 --out ui-frame.png
```

**Video verification:**
```bash
{baseDir}/scripts/frame.sh video.mp4 --out verify.jpg
```

## Source
SKILL.md: ` \openclaw\skills\video-frames\SKILL.md`
