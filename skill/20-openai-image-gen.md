# OpenAI Image Generation

**Location:** ` \openclaw\skills\openai-image-gen\`  
**Emoji:** 🖼️  
**Requirements:** `python3`, `OPENAI_API_KEY`  
**Homepage:** https://platform.openai.com/docs/api-reference/images

## Purpose
Batch-generate images via OpenAI Images API with random prompt sampler and HTML gallery.

## Run

**Basic usage:**
```bash
python3 {baseDir}/scripts/gen.py
open ~/Projects/tmp/openai-image-gen-*/index.html
```

## Models Supported

1. **GPT Image Models** - `gpt-image-1`, `gpt-image-1-mini`, `gpt-image-1.5`
2. **DALL-E 3** - `dall-e-3`
3. **DALL-E 2** - `dall-e-2`

## Common Options

### GPT Image Models

```bash
# Basic generation
python3 {baseDir}/scripts/gen.py --count 16 --model gpt-image-1

# Custom prompt
python3 {baseDir}/scripts/gen.py --prompt "ultra-detailed studio photo of a lobster astronaut" --count 4

# High quality landscape
python3 {baseDir}/scripts/gen.py --size 1536x1024 --quality high --out-dir ./out/images

# Transparent background, WebP output
python3 {baseDir}/scripts/gen.py --model gpt-image-1.5 --background transparent --output-format webp
```

**Supported sizes:**
- `1024x1024` (square, default)
- `1536x1024` (landscape)
- `1024x1536` (portrait)
- `auto`

**Quality levels:**
- `auto`, `high`, `medium`, `low` (default: `high`)

**Background:**
- `transparent`, `opaque`, `auto` (default)

**Output formats:**
- `png` (default), `jpeg`, `webp`

### DALL-E 3

```bash
# HD quality vivid style
python3 {baseDir}/scripts/gen.py --model dall-e-3 --quality hd --size 1792x1024 --style vivid

# Natural style
python3 {baseDir}/scripts/gen.py --model dall-e-3 --style natural --prompt "serene mountain landscape"
```

**Supported sizes:**
- `1024x1024` (default)
- `1792x1024` (landscape)
- `1024x1792` (portrait)

**Quality:**
- `hd` or `standard` (default)

**Style:**
- `vivid` (hyper-real, dramatic)
- `natural` (more natural looking)

**Note:** DALL-E 3 only generates 1 image at a time (count automatically limited to 1)

### DALL-E 2

```bash
# Multiple smaller images
python3 {baseDir}/scripts/gen.py --model dall-e-2 --size 512x512 --count 4
```

**Supported sizes:**
- `256x256`
- `512x512`
- `1024x1024` (default)

**Quality:** `standard` only

## Output Files

Generated in output directory:

1. **Images:** `*.png`, `*.jpeg`, or `*.webp` (depends on model/format)
2. **Prompts:** `prompts.json` (prompt → file mapping)
3. **Gallery:** `index.html` (thumbnail gallery viewer)

## Model Comparison

| Feature | GPT Image | DALL-E 3 | DALL-E 2 |
|---------|-----------|----------|----------|
| **Batch count** | ✅ Multiple | ❌ 1 only | ✅ Multiple |
| **Max size** | 1536x1024 | 1792x1024 | 1024x1024 |
| **Transparent BG** | ✅ Yes | ❌ No | ❌ No |
| **Output formats** | PNG/JPEG/WebP | PNG | PNG |
| **Quality levels** | 4 levels | HD/Standard | Standard only |
| **Style control** | Via prompt | Vivid/Natural | Via prompt |

## Best Practices

1. **GPT Image for batch work** - Multiple images, various formats
2. **DALL-E 3 for quality** - Best single images, HD option
3. **DALL-E 2 for speed** - Fast, cheap, smaller sizes
4. **Use transparent BG** for logos/assets (GPT Image only)
5. **Check output gallery** - `index.html` for easy review

## Source
SKILL.md: ` \openclaw\skills\openai-image-gen\SKILL.md`
