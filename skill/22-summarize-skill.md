# Summarize - URL/Video/File Summarization

**Location:** ` \openclaw\skills\summarize\`  
**Emoji:** 🧾  
**Requirements:** `summarize` CLI  
**Homepage:** https://summarize.sh

## Purpose
Fast CLI to summarize URLs, local files, and YouTube links. Great fallback for "transcribe this YouTube/video".

## Installation

```bash
brew install steipete/tap/summarize
```

## Trigger Phrases

Use this skill immediately when user asks:
- "use summarize.sh"
- "what's this link/video about?"
- "summarize this URL/article"
- "transcribe this YouTube/video"

## Quick Start

**Summarize URL:**
```bash
summarize "https://example.com" --model google/gemini-3-flash-preview
```

**Summarize local file:**
```bash
summarize "/path/to/file.pdf" --model google/gemini-3-flash-preview
```

**YouTube summary:**
```bash
summarize "https://youtu.be/dQw4w9WgXcQ" --youtube auto
```

## YouTube: Summary vs Transcript

**Best-effort transcript (URLs only):**
```bash
summarize "https://youtu.be/dQw4w9WgXcQ" --youtube auto --extract-only
```

**If transcript is huge:**
1. Return tight summary first
2. Ask which section/time range to expand

## Models & API Keys

Set API key for chosen provider:

| Provider | Environment Variable |
|----------|---------------------|
| **OpenAI** | `OPENAI_API_KEY` |
| **Anthropic** | `ANTHROPIC_API_KEY` |
| **xAI** | `XAI_API_KEY` |
| **Google** | `GEMINI_API_KEY` (also: `GOOGLE_GENERATIVE_AI_API_KEY`, `GOOGLE_API_KEY`) |

**Default model:** `google/gemini-3-flash-preview`

## Useful Flags

| Flag | Description |
|------|-------------|
| `--length short\|medium\|long\|xl\|xxl\|<chars>` | Summary length |
| `--max-output-tokens <count>` | Token limit |
| `--extract-only` | Extract text only (no summary) |
| `--json` | Machine-readable output |
| `--firecrawl auto\|off\|always` | Fallback extraction |
| `--youtube auto` | Apify fallback (needs `APIFY_API_TOKEN`) |

## Configuration

**Optional config:** `~/.summarize/config.json`
```json
{
  "model": "openai/gpt-5.2"
}
```

**Optional services:**
- `FIRECRAWL_API_KEY` - For blocked sites
- `APIFY_API_TOKEN` - For YouTube fallback

## Examples

**Short summary:**
```bash
summarize "https://example.com" --length short
```

**Extract only (no AI processing):**
```bash
summarize "https://example.com" --extract-only
```

**JSON output for scripting:**
```bash
summarize "https://example.com" --json
```

**YouTube transcript:**
```bash
summarize "https://youtu.be/VIDEO_ID" --youtube auto --extract-only
```

## Best Practices

1. **Use extract-only** for raw transcripts
2. **Start with short summaries** for quick overview
3. **Use JSON output** for scripting/automation
4. **Fallback services** (Firecrawl, Apify) for tricky content
5. **Gemini Flash** is fast & cheap default

## Source
SKILL.md: ` \openclaw\skills\summarize\SKILL.md`
