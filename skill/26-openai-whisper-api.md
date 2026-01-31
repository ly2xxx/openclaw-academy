# OpenAI Whisper API - Cloud Speech-to-Text

**Location:** ` \openclaw\skills\openai-whisper-api\`  
**Emoji:** ☁️  
**Requirements:** `curl`, `OPENAI_API_KEY`  
**Homepage:** https://platform.openai.com/docs/guides/speech-to-text

## Purpose
Transcribe audio via OpenAI's `/v1/audio/transcriptions` endpoint (cloud-based Whisper).

## Quick Start

**Basic transcription:**
```bash
{baseDir}/scripts/transcribe.sh /path/to/audio.m4a
```

**Defaults:**
- Model: `whisper-1`
- Output: `<input>.txt`

## Advanced Usage

**Specify model & output:**
```bash
{baseDir}/scripts/transcribe.sh /path/to/audio.ogg --model whisper-1 --out /tmp/transcript.txt
```

**Language hint:**
```bash
{baseDir}/scripts/transcribe.sh /path/to/audio.m4a --language en
```

**Context prompt (speaker names, etc.):**
```bash
{baseDir}/scripts/transcribe.sh /path/to/audio.m4a --prompt "Speaker names: Peter, Daniel"
```

**JSON output:**
```bash
{baseDir}/scripts/transcribe.sh /path/to/audio.m4a --json --out /tmp/transcript.json
```

## API Key Setup

**Environment variable:**
```bash
export OPENAI_API_KEY="sk-..."
```

**Or configure in OpenClaw config** (`~/.openclaw/openclaw.json`):
```json5
{
  "skills": {
    "openai-whisper-api": {
      "apiKey": "OPENAI_KEY_HERE"
    }
  }
}
```

## Supported Audio Formats

- M4A, MP3, MP4, MPEG, MPGA, OGG, WAV, WEBM
- Max file size: 25 MB

## vs Local Whisper

| Feature | Local Whisper | Whisper API |
|---------|---------------|-------------|
| **Cost** | Free (one-time install) | $0.006/min |
| **Speed** | Depends on hardware | Fast (cloud) |
| **Privacy** | Fully local | Sent to OpenAI |
| **Models** | Multiple sizes | whisper-1 only |
| **Setup** | Install models locally | API key only |
| **File limit** | No limit | 25 MB |

**Use local Whisper when:**
- Privacy concerns
- Batch processing many files
- No internet / offline work

**Use Whisper API when:**
- Fast, simple transcription needed
- Don't want to manage models
- Small/medium files only

## Best Practices

1. **Add language hint** for better accuracy
2. **Use prompt** for context (speaker names, domain jargon)
3. **JSON output** for structured data
4. **Check file size** before uploading (25 MB limit)
5. **Consider privacy** - audio sent to OpenAI

## Source
SKILL.md: ` \openclaw\skills\openai-whisper-api\SKILL.md`
