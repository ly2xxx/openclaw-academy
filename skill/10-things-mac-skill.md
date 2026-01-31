# Things 3 CLI (macOS)

**Location:** ` \openclaw\skills\things-mac\`  
**Emoji:** ✅  
**Requirements:** `things` CLI, macOS only  
**Homepage:** https://github.com/ossianhempel/things3-cli

## Purpose
Manage Things 3 via CLI: add/update projects+todos via URL scheme, read/search/list from local database.

## Installation

```bash
# Apple Silicon (recommended)
GOBIN=/opt/homebrew/bin go install github.com/ossianhempel/things3-cli/cmd/things@latest
```

## Setup

**Full Disk Access Required:**
Grant **Full Disk Access** to:
- **Terminal** for manual runs
- **OpenClaw.app** for gateway runs

**Optional Environment:**
- `THINGSDB` - Point to `ThingsData-*` folder
- `THINGS_AUTH_TOKEN` - Avoid passing `--auth-token` for updates

## Read-Only Operations (DB)

```bash
things inbox --limit 50
things today
things upcoming
things search "query"
things projects
things areas
things tags
```

## Write Operations (URL Scheme)

### Add Todo

**Basic:**
```bash
things add "Buy milk"
```

**With options:**
```bash
things add "Buy milk" --notes "2% + bananas"
things add "Book flights" --list "Travel"
things add "Pack charger" --list "Travel" --heading "Before"
things add "Call dentist" --tags "health,phone"
```

**Scheduling:**
```bash
things add "Title" --when today --deadline 2026-01-02
```

**Checklist:**
```bash
things add "Trip prep" --checklist-item "Passport" --checklist-item "Tickets"
```

**Multi-line from STDIN:**
```bash
cat <<'EOF' | things add -
Title line
Notes line 1
Notes line 2
EOF
```

**Bring Things to front:**
```bash
things --foreground add "Title"
```

### Update Todo (Requires Auth Token)

**First, get the ID:**
```bash
things search "milk" --limit 5
# Note the UUID column
```

**Update operations:**
```bash
# Title
things update --id <UUID> --auth-token <TOKEN> "New title"

# Notes (replace)
things update --id <UUID> --auth-token <TOKEN> --notes "New notes"

# Notes (append/prepend)
things update --id <UUID> --auth-token <TOKEN> --append-notes "..."
things update --id <UUID> --auth-token <TOKEN> --prepend-notes "..."

# Move to different list/heading
things update --id <UUID> --auth-token <TOKEN> --list "Travel" --heading "Before"

# Tags (replace/add)
things update --id <UUID> --auth-token <TOKEN> --tags "a,b"
things update --id <UUID> --auth-token <TOKEN> --add-tags "a,b"

# Mark complete/canceled
things update --id <UUID> --auth-token <TOKEN> --completed
things update --id <UUID> --auth-token <TOKEN> --canceled
```

### Delete Todo

**Not supported** - Use alternatives:
1. Delete manually in Things UI
2. Mark as `--completed` or `--canceled`

## Safe Preview

Always preview before executing:
```bash
things --dry-run add "Title"
things --dry-run update --id <UUID> --auth-token <TOKEN> --completed
```

Prints URL without opening Things.

## Best Practices

1. **Use --dry-run** for new/complex operations
2. **Search first** to get UUID for updates
3. **Set THINGS_AUTH_TOKEN** to avoid passing token each time
4. **Grant Full Disk Access** to avoid permission errors
5. **macOS only** - This skill won't work on other platforms

## Source
SKILL.md: ` \openclaw\skills\things-mac\SKILL.md`
