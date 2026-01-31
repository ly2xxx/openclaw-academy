# Gemini CLI

**Location:** ` \openclaw\skills\gemini\`  
**Emoji:** ♊️  
**Requirements:** `gemini` CLI  
**Homepage:** https://ai.google.dev/

## Purpose
Gemini CLI for one-shot Q&A, summaries, and generation.

## Installation

```bash
brew install gemini-cli
```

## Quick Start

**Basic usage:**
```bash
gemini "Answer this question..."
```

**Specify model:**
```bash
gemini --model gemini-2.5-pro "Prompt..."
```

**JSON output:**
```bash
gemini --output-format json "Return JSON"
```

## Extensions

**List available extensions:**
```bash
gemini --list-extensions
```

**Manage extensions:**
```bash
gemini extensions <command>
```

## Authentication

If auth required, run `gemini` once interactively and follow login flow.

## Best Practices

1. **One-shot mode preferred** - avoid interactive mode
2. **Positional prompt** - pass prompt directly as argument
3. **Avoid --yolo** for safety
4. **Use --output-format** for structured output

## Source
SKILL.md: ` \openclaw\skills\gemini\SKILL.md`
