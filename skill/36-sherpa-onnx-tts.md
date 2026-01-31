# Sherpa-ONNX TTS - Offline Text-to-Speech

**Location:** ` \openclaw\skills\sherpa-onnx-tts\`  
**Emoji:** 🗣️  
**Requirements:** sherpa-onnx runtime + models  
**OS Support:** macOS, Linux, Windows

## Purpose
Local text-to-speech using sherpa-onnx offline CLI (no cloud, no API key).

## Installation

### 1. Download Runtime

**macOS:**
```
https://github.com/k2-fsa/sherpa-onnx/releases/download/v1.12.23/sherpa-onnx-v1.12.23-osx-universal2-shared.tar.bz2
```

**Linux x64:**
```
https://github.com/k2-fsa/sherpa-onnx/releases/download/v1.12.23/sherpa-onnx-v1.12.23-linux-x64-shared.tar.bz2
```

**Windows x64:**
```
https://github.com/k2-fsa/sherpa-onnx/releases/download/v1.12.23/sherpa-onnx-v1.12.23-win-x64-shared.tar.bz2
```

Extract to: `~/.openclaw/tools/sherpa-onnx-tts/runtime`

### 2. Download Voice Model

**Piper en_US lessac (high quality):**
```
https://github.com/k2-fsa/sherpa-onnx/releases/download/tts-models/vits-piper-en_US-lessac-high.tar.bz2
```

Extract to: `~/.openclaw/tools/sherpa-onnx-tts/models/vits-piper-en_US-lessac-high`

### 3. Configure OpenClaw

Update `~/.openclaw/openclaw.json`:

```json5
{
  "skills": {
    "entries": {
      "sherpa-onnx-tts": {
        "env": {
          "SHERPA_ONNX_RUNTIME_DIR": "~/.openclaw/tools/sherpa-onnx-tts/runtime",
          "SHERPA_ONNX_MODEL_DIR": "~/.openclaw/tools/sherpa-onnx-tts/models/vits-piper-en_US-lessac-high"
        }
      }
    }
  }
}
```

## Usage

### Basic TTS

```bash
{baseDir}/bin/sherpa-onnx-tts -o ./tts.wav "Hello from local TTS."
```

### Add to PATH

```bash
export PATH="{baseDir}/bin:$PATH"
sherpa-onnx-tts -o output.wav "Your text here"
```

### Windows

```bash
node {baseDir}\\bin\\sherpa-onnx-tts -o tts.wav "Hello from local TTS."
```

## Advanced Options

**Specify model file:**
```bash
sherpa-onnx-tts --model-file custom.onnx -o output.wav "Text"
```

**Environment override:**
```bash
export SHERPA_ONNX_MODEL_FILE=custom.onnx
sherpa-onnx-tts -o output.wav "Text"
```

**Custom tokens/data:**
```bash
sherpa-onnx-tts --tokens-file tokens.txt --data-dir ./data -o output.wav "Text"
```

## Available Voices

Browse more models at:
```
https://github.com/k2-fsa/sherpa-onnx/releases?q=tts-models
```

**Popular options:**
- `vits-piper-en_US-lessac-high` (high quality English)
- `vits-piper-en_US-libritts-high` (LibriTTS English)
- Multi-language models available

## vs Cloud TTS (SAG/ElevenLabs)

| Feature | Sherpa-ONNX | ElevenLabs (SAG) |
|---------|-------------|------------------|
| **Cost** | Free (one-time download) | API costs per character |
| **Privacy** | Fully local | Sent to cloud |
| **Quality** | Good | Excellent |
| **Voices** | Limited selection | Many voices + custom |
| **Latency** | Fast (local) | Network dependent |
| **Offline** | ✅ Yes | ❌ No |

**Use Sherpa-ONNX when:**
- Privacy concerns
- Offline operation
- High volume / cost sensitive
- Acceptable quality threshold

**Use ElevenLabs when:**
- Highest quality needed
- Variety of voices
- Emotional expressiveness
- Custom voice cloning

## Best Practices

1. **Download models once** - reuse locally
2. **Multiple voices** - download different models for variety
3. **Batch processing** - efficient for multiple files
4. **Test quality** - try different models for best fit

## Troubleshooting

**Runtime not found:**
- Check `SHERPA_ONNX_RUNTIME_DIR` path
- Verify extraction completed

**Model not found:**
- Check `SHERPA_ONNX_MODEL_DIR` path
- Ensure .onnx files present in model directory

**Multiple .onnx files:**
- Set `SHERPA_ONNX_MODEL_FILE` or use `--model-file`

## Source
SKILL.md: ` \openclaw\skills\sherpa-onnx-tts\SKILL.md`
