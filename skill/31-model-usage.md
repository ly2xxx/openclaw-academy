# Model Usage - CodexBar Cost Tracking

**Location:** ` \openclaw\skills\model-usage\`  
**Emoji:** 📊  
**Requirements:** `codexbar` CLI, macOS  
**OS Support:** macOS (Linux support TODO)

## Purpose
Get per-model usage and cost data from CodexBar's local logs for Codex or Claude.

## Installation

```bash
brew install --cask steipete/tap/codexbar
```

## Quick Start

**Current model (most recent):**
```bash
python {baseDir}/scripts/model_usage.py --provider codex --mode current
```

**All models summary:**
```bash
python {baseDir}/scripts/model_usage.py --provider codex --mode all
```

**Claude usage:**
```bash
python {baseDir}/scripts/model_usage.py --provider claude --mode all --format json --pretty
```

## Current Model Logic

1. Uses most recent daily row with `modelBreakdowns`
2. Picks model with highest cost in that row
3. Falls back to last entry in `modelsUsed` if no breakdowns
4. Override with `--model <name>` for specific model

## Input Options

**Default (auto-fetch):**
```bash
python {baseDir}/scripts/model_usage.py --provider codex --mode current
```

**From file:**
```bash
codexbar cost --provider codex --format json > /tmp/cost.json
python {baseDir}/scripts/model_usage.py --input /tmp/cost.json --mode all
```

**From stdin:**
```bash
cat /tmp/cost.json | python {baseDir}/scripts/model_usage.py --input - --mode current
```

## Output Formats

**Text (default):**
```bash
python {baseDir}/scripts/model_usage.py --provider codex --mode all
```

**JSON (pretty):**
```bash
python {baseDir}/scripts/model_usage.py --provider codex --mode all --format json --pretty
```

## Examples

**Codex daily cost:**
```bash
python {baseDir}/scripts/model_usage.py --provider codex --mode current
```

**Claude model breakdown:**
```bash
python {baseDir}/scripts/model_usage.py --provider claude --mode all --format json
```

**Specific model override:**
```bash
python {baseDir}/scripts/model_usage.py --provider codex --mode current --model gpt-5.2-codex
```

## Notes

1. **Values are cost-only** - tokens not split by model in CodexBar output
2. **macOS only** currently (Linux support pending)
3. **Requires CodexBar** to be running and tracking usage
4. **Local data** - reads from CodexBar's local database

## Use Cases

**Track AI development costs:**
```bash
python {baseDir}/scripts/model_usage.py --provider codex --mode all
```

**Compare Claude vs Codex:**
```bash
python {baseDir}/scripts/model_usage.py --provider codex --mode all
python {baseDir}/scripts/model_usage.py --provider claude --mode all
```

**Daily cost monitoring:**
```bash
python {baseDir}/scripts/model_usage.py --provider codex --mode current --format json | jq '.cost'
```

## References

See `references/codexbar-cli.md` for CLI flags and cost JSON fields.

## Source
SKILL.md: ` \openclaw\skills\model-usage\SKILL.md`
