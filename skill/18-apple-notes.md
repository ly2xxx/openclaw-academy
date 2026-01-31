# Apple Notes CLI (memo)

**Location:** ` \openclaw\skills\apple-notes\`  
**Emoji:** 📝  
**Requirements:** `memo` CLI, macOS only  
**Homepage:** https://github.com/antoniorodr/memo

## Purpose
Manage Apple Notes via CLI - create, view, edit, delete, search, move, and export notes.

## Installation

```bash
brew tap antoniorodr/memo
brew install antoniorodr/memo/memo
```

**Alternative (pip):**
```bash
pip install .  # After cloning repo
```

## Setup

**Permissions:** Grant Automation access to Notes.app when prompted.  
Location: System Settings > Privacy & Security > Automation

## Common Operations

### View Notes

```bash
# List all notes
memo notes

# Filter by folder
memo notes -f "Folder Name"

# Search notes (fuzzy)
memo notes -s "query"
```

### Create Notes

```bash
# Interactive add (opens editor)
memo notes -a

# Quick add with title
memo notes -a "Note Title"
```

### Edit Notes

```bash
# Edit existing (interactive selection)
memo notes -e
```

### Delete Notes

```bash
# Delete (interactive selection)
memo notes -d
```

### Move Notes

```bash
# Move to folder (interactive selection)
memo notes -m
```

### Export Notes

```bash
# Export to HTML/Markdown (interactive selection)
memo notes -ex
```

Uses Mistune for markdown processing.

## Limitations

1. **Cannot edit notes** containing images or attachments
2. **Interactive prompts** require terminal access
3. **macOS only**

## Best Practices

1. **Grant permissions** in System Settings for automation
2. **Use search** for quick note lookup
3. **Export** for backups or migration
4. **Plain text notes** work best (no images/attachments)

## Source
SKILL.md: ` \openclaw\skills\apple-notes\SKILL.md`
