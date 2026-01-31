# ClawHub - Skill Marketplace CLI

**Location:** ` \openclaw\skills\clawhub\`  
**Requirements:** `clawhub` CLI  
**Registry:** https://clawhub.com

## Purpose
Search, install, update, and publish agent skills from ClawHub marketplace.

## Installation

```bash
npm install -g clawhub
```

## Authentication (for Publishing)

```bash
clawhub login
clawhub whoami
```

## Common Commands

### Search Skills

```bash
clawhub search "postgres backups"
```

### Install Skill

```bash
# Latest version
clawhub install my-skill

# Specific version
clawhub install my-skill --version 1.2.3
```

### Update Skills

```bash
# Update single skill
clawhub update my-skill

# Update to specific version
clawhub update my-skill --version 1.2.3

# Update all skills
clawhub update --all

# Force update
clawhub update my-skill --force

# Non-interactive force update all
clawhub update --all --no-input --force
```

### List Installed

```bash
clawhub list
```

### Publish Skill

```bash
clawhub publish ./my-skill \
  --slug my-skill \
  --name "My Skill" \
  --version 1.2.0 \
  --changelog "Fixes + docs"
```

## Update Mechanism

**Hash-based matching:**
1. Hashes local skill files
2. Resolves matching version on ClawHub
3. Upgrades to latest (unless `--version` specified)

## Configuration

### Registry

**Default:** `https://clawhub.com`

**Override:**
```bash
export CLAWHUB_REGISTRY="https://custom-registry.com"
# Or use --registry flag
clawhub search "query" --registry https://custom-registry.com
```

### Working Directory

**Default:** Current directory (falls back to OpenClaw workspace)

**Override:**
```bash
export CLAWHUB_WORKDIR="/path/to/skills"
# Or use --workdir flag
clawhub install my-skill --workdir /path/to/skills
```

**Install directory:** `./skills` (override with `--dir`)

## Best Practices

1. **Search before building** - skill may already exist
2. **Version pinning** for stability
3. **Test before publishing** - validate skill works
4. **Semantic versioning** - MAJOR.MINOR.PATCH
5. **Meaningful changelogs** - help users understand updates

## Common Workflows

**Discover & install:**
```bash
clawhub search "notion"
clawhub install notion-sync
```

**Update all skills:**
```bash
clawhub update --all
```

**Publish new skill:**
```bash
clawhub login
clawhub publish ./my-awesome-skill \
  --slug my-awesome-skill \
  --name "My Awesome Skill" \
  --version 1.0.0 \
  --changelog "Initial release"
```

**Force reinstall:**
```bash
clawhub update my-skill --force
```

## Publishing Checklist

1. ✅ Skill folder structure correct
2. ✅ SKILL.md complete with metadata
3. ✅ Test skill locally
4. ✅ Login to ClawHub
5. ✅ Choose unique slug
6. ✅ Semantic version number
7. ✅ Write changelog
8. ✅ Publish!

## Troubleshooting

**Authentication failed:**
```bash
clawhub login
```

**Skill not found:**
- Check spelling
- Search on https://clawhub.com

**Update fails:**
- Try `--force` flag
- Check network connection
- Verify registry URL

**Publish fails:**
- Ensure logged in
- Check slug uniqueness
- Validate SKILL.md format

## Source
SKILL.md: ` \openclaw\skills\clawhub\SKILL.md`
