# Obsidian Skill

**Location:** ` \openclaw\skills\obsidian\`  
**Emoji:** 💎  
**Requirements:** `obsidian-cli` CLI tool  
**Homepage:** https://help.obsidian.md

## Purpose
Work with Obsidian vaults (plain Markdown notes) and automate via obsidian-cli.

## Obsidian Vault Structure

**Vault = normal folder on disk**

Typical structure:
- **Notes:** `*.md` (plain text Markdown; edit with any editor)
- **Config:** `.obsidian/` (workspace + plugin settings; usually don't touch)
- **Canvases:** `*.canvas` (JSON)
- **Attachments:** Images/PDFs in configured folder

## Finding Active Vaults

**Source of truth:** `~/Library/Application Support/obsidian/obsidian.json`

`obsidian-cli` resolves vaults from this file. Vault name is typically the **folder name** (path suffix).

**Quick check:**
- If default set: `obsidian-cli print-default --path-only`
- Otherwise: Read `obsidian.json` and use vault with `"open": true`

**Notes:**
- Multiple vaults common (iCloud vs ~/Documents, work/personal)
- Don't guess; read config
- Avoid hardcoded vault paths in scripts

## Installation

**macOS:**
```bash
brew install yakitrak/yakitrak/obsidian-cli
```

## obsidian-cli Quick Reference

### Set Default Vault
```bash
obsidian-cli set-default "<vault-folder-name>"
obsidian-cli print-default
obsidian-cli print-default --path-only
```

### Search
```bash
# Search note names
obsidian-cli search "query"

# Search inside notes (shows snippets + lines)
obsidian-cli search-content "query"
```

### Create
```bash
obsidian-cli create "Folder/New note" --content "..." --open
```

**Requirements:**
- Obsidian URI handler (`obsidian://…`) working (Obsidian installed)
- Avoid creating under hidden dot-folders (e.g., `.something/...`) - Obsidian may refuse

### Move/Rename (Safe Refactor)
```bash
obsidian-cli move "old/path/note" "new/path/note"
```

**Key benefit:** Updates `[[wikilinks]]` and common Markdown links across the vault!  
This is the main win vs plain `mv`.

### Delete
```bash
obsidian-cli delete "path/note"
```

## Direct Edits

**Prefer direct edits when appropriate:**
- Open the `.md` file directly and change it
- Obsidian will pick up changes automatically

## Best Practices

1. **Don't hardcode vault paths** - Read from `obsidian.json` or use `print-default`
2. **Use `move` not `mv`** - Preserves links across vault
3. **Respect `.obsidian/`** - Don't modify config files from scripts
4. **Plain Markdown** - Notes are just text files, edit with any tool

## Source
SKILL.md: ` \openclaw\skills\obsidian\SKILL.md`
