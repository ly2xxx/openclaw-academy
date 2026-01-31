# Skill Creator - How to Build OpenClaw Skills

**Location:** ` \openclaw\skills\skill-creator\`  
**Purpose:** Create or update AgentSkills - modular packages that extend OpenClaw's capabilities

## What Skills Provide

1. **Specialized workflows** - Multi-step procedures for specific domains
2. **Tool integrations** - Instructions for working with specific file formats or APIs
3. **Domain expertise** - Company-specific knowledge, schemas, business logic
4. **Bundled resources** - Scripts, references, and assets for complex tasks

## Core Principles

### 1. Concise is Key
- Context window is a public good shared with everything else
- Default assumption: OpenClaw is already very smart
- Only add context it doesn't already have
- Challenge each piece: "Does this justify its token cost?"

### 2. Set Appropriate Degrees of Freedom
- **High freedom (text instructions)**: Multiple valid approaches, context-dependent decisions
- **Medium freedom (pseudocode with parameters)**: Preferred patterns, some variation okay
- **Low freedom (specific scripts)**: Fragile operations, consistency critical, specific sequences

### 3. Progressive Disclosure (3-level loading)
1. **Metadata** (name + description) - Always in context (~100 words)
2. **SKILL.md body** - When skill triggers (<5k words)
3. **Bundled resources** - As needed (unlimited)

## Skill Structure

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name, description)
│   └── Markdown instructions
└── Bundled Resources (optional)
    ├── scripts/          - Executable code
    ├── references/       - Documentation
    └── assets/           - Output files
```

### SKILL.md Components

**Frontmatter (YAML):**
- `name`: Skill name
- `description`: PRIMARY TRIGGERING MECHANISM
  - What the skill does AND when to use it
  - All "when to use" info goes here (body is loaded AFTER triggering)

**Body (Markdown):**
- Instructions for using the skill
- References to bundled resources

## Bundled Resources

### scripts/
- Executable code (Python/Bash/etc.)
- **When:** Repeatedly rewritten code or need deterministic reliability
- **Example:** `rotate_pdf.py` for PDF rotation
- **Benefits:** Token efficient, deterministic, may execute without loading

### references/
- Documentation loaded into context as needed
- **When:** Documentation OpenClaw should reference while working
- **Examples:** API docs, schemas, policies, workflow guides
- **Best practice:** Large files (>10k words) need grep search patterns in SKILL.md

### assets/
- Files used in output (NOT loaded into context)
- **When:** Files needed in final output
- **Examples:** Templates, images, boilerplate code, fonts
- **Use cases:** HTML/React templates, brand assets, sample documents

## Creation Process

1. **Understand** - Get concrete examples of skill usage
2. **Plan** - Identify reusable scripts, references, assets
3. **Initialize** - Run `scripts/init_skill.py <skill-name> --path <output-dir>`
4. **Edit** - Implement resources and write SKILL.md
5. **Package** - Run `scripts/package_skill.py <path/to/skill-folder>`
6. **Iterate** - Test and improve based on usage

## Naming Conventions

- Lowercase letters, digits, hyphens only
- Under 64 characters
- Verb-led phrases describing the action
- Example: "Plan Mode" → `plan-mode`

## Validation & Packaging

```bash
# Initialize new skill
scripts/init_skill.py my-skill --path skills/public --resources scripts,references

# Package (auto-validates first)
scripts/package_skill.py <path/to/skill-folder>
```

Creates `my-skill.skill` (zip file with .skill extension)

## Key Resources

- **Multi-step processes**: `references/workflows.md`
- **Output formats**: `references/output-patterns.md`

## Source
SKILL.md: ` \openclaw\skills\skill-creator\SKILL.md`
Scripts: ` \openclaw\scripts\init_skill.py`, `package_skill.py`
