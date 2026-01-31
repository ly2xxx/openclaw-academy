# SAG - ElevenLabs Text-to-Speech

**Location:** ` \openclaw\skills\sag\`  
**Emoji:** 🗣️  
**Requirements:** `sag` CLI, `ELEVENLABS_API_KEY` env var  
**Homepage:** https://sag.sh

## Purpose
ElevenLabs text-to-speech with mac-style `say` UX and local playback.

## Installation

```bash
brew install steipete/tap/sag
```

## API Key Setup

Required (pick one):
- `ELEVENLABS_API_KEY` (preferred)
- `SAG_API_KEY` (also supported)

## Quick Start

```bash
sag "Hello there"
sag speak -v "Roger" "Hello"
sag voices
sag prompting  # Model-specific tips
```

## Models

- **Default:** `eleven_v3` (expressive)
- **Stable:** `eleven_multilingual_v2`
- **Fast:** `eleven_flash_v2_5`

## Pronunciation & Delivery

### Fixing Pronunciation
1. **First try:** Respell (e.g., "key-note"), add hyphens, adjust casing
2. **Numbers/URLs:** `--normalize auto` (or `off` if it harms names)
3. **Language bias:** `--lang en|de|fr|...` to guide normalization

### SSML Support
- **v3:** No `<break>` support - Use `[pause]`, `[short pause]`, `[long pause]`
- **v2/v2.5:** SSML `<break time="1.5s" />` supported (no `<phoneme>`)

## v3 Audio Tags (at start of line)

**Emotions & Tone:**
- `[whispers]`, `[shouts]`, `[sings]`
- `[laughs]`, `[starts laughing]`, `[sighs]`, `[exhales]`
- `[sarcastic]`, `[curious]`, `[excited]`, `[crying]`, `[mischievously]`

**Example:**
```bash
sag "[whispers] keep this quiet. [short pause] ok?"
```

## Voice Defaults

Set default voice:
- `ELEVENLABS_VOICE_ID` or `SAG_VOICE_ID`

**Clawd's default voice:** `lj2rcrvANS3gaWWnczSX` (or `-v Clawd`)

## Chat Voice Responses

When user asks for "voice" reply (e.g., "crazy scientist voice"):

```bash
# Generate audio
sag -v Clawd -o /tmp/voice-reply.mp3 "Your message here"

# Include in reply:
# MEDIA:/tmp/voice-reply.mp3
```

### Voice Character Tips

- **Crazy scientist:** `[excited]` tags, dramatic `[short pause]`, vary intensity
- **Calm:** `[whispers]` or slower pacing
- **Dramatic:** `[sings]` or `[shouts]` sparingly

## Best Practices

1. **Confirm voice + speaker before long output**
2. **Preview with shorter text first**
3. **Use audio tags at line start for best results**

## Source
SKILL.md: ` \openclaw\skills\sag\SKILL.md`
