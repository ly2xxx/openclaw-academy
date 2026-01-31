# Nano Banana Pro - Gemini Image Generation

**Location:** ` \openclaw\skills\nano-banana-pro\`  
**Emoji:** 🍌  
**Requirements:** `uv`, `GEMINI_API_KEY`  
**Homepage:** https://ai.google.dev/

## Purpose
Generate or edit images via Gemini 3 Pro Image (Nano Banana Pro).

## Setup

**API Key:**
```bash
export GEMINI_API_KEY="your-key"
```

**Or in OpenClaw config** (`~/.openclaw/openclaw.json`):
```json5
{
  "skills": {
    "nano-banana-pro": {
      "apiKey": "your-key"
    }
  }
}
```

## Commands

### Generate Image

```bash
uv run {baseDir}/scripts/generate_image.py \
  --prompt "your image description" \
  --filename "output.png" \
  --resolution 1K
```

### Edit Single Image

```bash
uv run {baseDir}/scripts/generate_image.py \
  --prompt "edit instructions" \
  --filename "output.png" \
  -i "/path/input.png" \
  --resolution 2K
```

### Multi-Image Composition

```bash
uv run {baseDir}/scripts/generate_image.py \
  --prompt "combine these into one scene" \
  --filename "output.png" \
  -i img1.png \
  -i img2.png \
  -i img3.png
```

**Note:** Up to 14 images supported.

## Resolutions

| Resolution | Description |
|------------|-------------|
| `1K` | Default (1024x1024 or similar) |
| `2K` | Higher quality |
| `4K` | Maximum quality |

## Best Practices

1. **Use timestamps in filenames:** `2026-01-31-14-30-output.png`
2. **Don't read image back** - just report saved path
3. **MEDIA: line** is printed by script for auto-attach on chat providers
4. **Multi-image limit** is 14 images max

## Examples

**Generate landscape:**
```bash
uv run {baseDir}/scripts/generate_image.py \
  --prompt "serene mountain landscape at sunset, photorealistic" \
  --filename "landscape.png" \
  --resolution 2K
```

**Edit existing image:**
```bash
uv run {baseDir}/scripts/generate_image.py \
  --prompt "change sky to dramatic sunset, add birds" \
  --filename "edited.png" \
  -i original.png \
  --resolution 1K
```

**Combine multiple images:**
```bash
uv run {baseDir}/scripts/generate_image.py \
  --prompt "create a collage from these vacation photos" \
  --filename "collage.png" \
  -i photo1.jpg \
  -i photo2.jpg \
  -i photo3.jpg \
  --resolution 2K
```

**Quick generation:**
```bash
uv run {baseDir}/scripts/generate_image.py \
  --prompt "futuristic robot assistant" \
  --filename "robot.png"
```

## Output

Script prints:
```
MEDIA:/path/to/output.png
```

OpenClaw auto-attaches this on supported chat providers (WhatsApp, Discord, etc.).

## Notes

1. **Don't verify image** - trust the script output
2. **Filename best practice:** Include timestamp and description
3. **Multi-image composition** is powerful for combining elements
4. **Resolution trade-off:** Higher = better quality but slower

## Source
SKILL.md: ` \openclaw\skills\nano-banana-pro\SKILL.md`
