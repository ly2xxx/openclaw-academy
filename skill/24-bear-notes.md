# Bear Notes CLI (grizzly)

**Location:** ` \openclaw\skills\bear-notes\`  
**Emoji:** 🐻  
**Requirements:** `grizzly` CLI, Bear app (macOS)  
**Homepage:** https://bear.app

## Purpose
Create, search, and manage Bear notes via CLI.

## Installation

```bash
go install github.com/tylerwince/grizzly/cmd/grizzly@latest
```

## Getting Bear Token

For operations requiring token (add-text, tags, open-note --selected):

1. Open Bear → Help → API Token → Copy Token
2. Save it:
   ```bash
   mkdir -p ~/.config/grizzly
   echo "YOUR_TOKEN" > ~/.config/grizzly/token
   ```

## Common Commands

### Create Note

```bash
# From stdin
echo "Note content here" | grizzly create --title "My Note" --tag work

# Empty note with tags
grizzly create --title "Quick Note" --tag inbox < /dev/null
```

### Open/Read Note

```bash
# By ID
grizzly open-note --id "NOTE_ID" --enable-callback --json
```

### Append to Note

```bash
echo "Additional content" | grizzly add-text --id "NOTE_ID" --mode append --token-file ~/.config/grizzly/token
```

### List Tags

```bash
grizzly tags --enable-callback --json --token-file ~/.config/grizzly/token
```

### Search Notes

```bash
# Via tag
grizzly open-tag --name "work" --enable-callback --json
```

## Common Flags

| Flag | Description |
|------|-------------|
| `--dry-run` | Preview URL without executing |
| `--print-url` | Show x-callback-url |
| `--enable-callback` | Wait for Bear's response (needed for reading data) |
| `--json` | Output as JSON (with callbacks) |
| `--token-file PATH` | Path to Bear API token file |

## Configuration

**Priority order:**
1. CLI flags
2. Environment variables (`GRIZZLY_TOKEN_FILE`, `GRIZZLY_CALLBACK_URL`, `GRIZZLY_TIMEOUT`)
3. `.grizzly.toml` in current directory
4. `~/.config/grizzly/config.toml`

**Example** `~/.config/grizzly/config.toml`:
```toml
token_file = "~/.config/grizzly/token"
callback_url = "http://127.0.0.1:42123/success"
timeout = "5s"
```

## Notes

1. **Bear must be running** for commands to work
2. **Note IDs** are Bear's internal identifiers (visible in note info or via callbacks)
3. **Use --enable-callback** when reading data back from Bear
4. **Some operations require token** (add-text, tags, open-note --selected)

## Best Practices

1. **Store token** in config file for convenience
2. **Use JSON output** for scripting
3. **Enable callbacks** for data retrieval
4. **Tag notes** for better organization
5. **Dry-run** for testing commands

## Source
SKILL.md: ` \openclaw\skills\bear-notes\SKILL.md`
