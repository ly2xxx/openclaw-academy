# Apple Reminders CLI (remindctl)

**Location:** ` \openclaw\skills\apple-reminders\`  
**Emoji:** ⏰  
**Requirements:** `remindctl` CLI, macOS only  
**Homepage:** https://github.com/steipete/remindctl

## Purpose
Manage Apple Reminders via CLI - list, add, edit, complete, delete with date filters and JSON output.

## Installation

```bash
brew install steipete/tap/remindctl
```

**From source:**
```bash
pnpm install && pnpm build
# Binary at ./bin/remindctl
```

## Setup

**Check permissions:**
```bash
remindctl status
```

**Request access:**
```bash
remindctl authorize
```

Grant Reminders permission when prompted:  
System Settings → Privacy & Security → Reminders

## View Reminders

```bash
# Default (today)
remindctl

# Specific views
remindctl today
remindctl tomorrow
remindctl week
remindctl overdue
remindctl upcoming
remindctl completed
remindctl all

# Specific date
remindctl 2026-01-04
```

## Manage Lists

```bash
# List all lists
remindctl list

# Show specific list
remindctl list Work

# Create list
remindctl list Projects --create

# Rename list
remindctl list Work --rename Office

# Delete list
remindctl list Work --delete
```

## Create Reminders

```bash
# Quick add
remindctl add "Buy milk"

# With list + due date
remindctl add --title "Call mom" --list Personal --due tomorrow
```

## Edit Reminders

```bash
# Edit title/due date
remindctl edit 1 --title "New title" --due 2026-01-04
```

## Complete Reminders

```bash
# Complete by ID
remindctl complete 1 2 3
```

## Delete Reminders

```bash
# Delete by ID (force required)
remindctl delete 4A83 --force
```

## Output Formats

```bash
# JSON (for scripting)
remindctl today --json

# Plain TSV
remindctl today --plain

# Counts only
remindctl today --quiet
```

## Date Formats

Accepted by `--due` and date filters:
- `today`, `tomorrow`, `yesterday`
- `YYYY-MM-DD`
- `YYYY-MM-DD HH:mm`
- ISO 8601 (`2026-01-04T12:34:56Z`)

## Notes

1. **macOS only**
2. **SSH access:** Grant permissions on the Mac running the command
3. **Access denied?** Enable in System Settings → Privacy & Security → Reminders

## Best Practices

1. **Use JSON output** for scripting
2. **Specific views** for focused lists (today, overdue)
3. **Date shortcuts** (today, tomorrow) for quick entry
4. **Force flag** required for deletions (safety)

## Source
SKILL.md: ` \openclaw\skills\apple-reminders\SKILL.md`
