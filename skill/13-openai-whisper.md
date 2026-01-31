# OpenAI Whisper - Local Speech-to-Text

**Location:** ` \openclaw\skills\openai-whisper\`  
**Emoji:** 🎙️  
**Requirements:** `whisper` CLI (no API key needed!)  
**Homepage:** https://openai.com/research/whisper

## Purpose
Local speech-to-text transcription with OpenAI's Whisper model.

## Installation

```bash
brew install openai-whisper
```

## Quick Start

**Basic transcription:**
```bash
whisper /path/audio.mp3 --model medium --output_format txt --output_dir .
```

**Translation to English:**
```bash
whisper /path/audio.m4a --task translate --output_format srt
```

## Models

Models download to `~/.cache/whisper` on first run.

| Model | Speed | Accuracy | Use Case |
|-------|-------|----------|----------|
| `tiny` | Fastest | Lowest | Quick drafts |
| `base` | Fast | Low | Simple audio |
| `small` | Medium | Medium | General use |
| `medium` | Slower | High | Quality transcripts |
| `large` | Slowest | Highest | Professional |
| `turbo` | Fast | High | **Default on this install** |

## Output Formats

- `txt` - Plain text
- `srt` - Subtitles with timestamps
- `vtt` - WebVTT subtitles
- `json` - Detailed JSON with segments

## Best Practices

1. **Start with turbo/medium** - good balance of speed/accuracy
2. **Use smaller models** for quick checks
3. **Use larger models** for final transcripts
4. **Local processing** - no API costs, privacy-preserving

## Source
SKILL.md: ` \openclaw\skills\openai-whisper\SKILL.md`
