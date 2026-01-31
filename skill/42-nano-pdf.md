# Nano PDF - Natural Language PDF Editing

**Location:** ` \openclaw\skills\nano-pdf\`  
**Emoji:** 📄  
**Requirements:** `nano-pdf` CLI  
**Homepage:** https://pypi.org/project/nano-pdf/

## Purpose
Edit PDFs with natural-language instructions using AI.

## Installation

```bash
uv tool install nano-pdf
```

**Or via pip:**
```bash
pip install nano-pdf
```

## Quick Start

```bash
nano-pdf edit deck.pdf 1 "Change the title to 'Q3 Results' and fix the typo in the subtitle"
```

## Command Format

```bash
nano-pdf edit <pdf-file> <page-number> "<instruction>"
```

**Parameters:**
- `<pdf-file>` - Path to PDF file
- `<page-number>` - Page to edit
- `<instruction>` - Natural language edit instruction

## Page Numbering

**⚠️ Important:** Page numbers may be 0-based or 1-based depending on tool version.

**If output looks off by one:**
- Try the other numbering scheme
- Page 1 might be index 0 or index 1

## Examples

**Change title:**
```bash
nano-pdf edit presentation.pdf 1 "Change the title to 'Annual Report 2026'"
```

**Fix typo:**
```bash
nano-pdf edit document.pdf 5 "Fix the typo in the third paragraph - change 'recieve' to 'receive'"
```

**Update date:**
```bash
nano-pdf edit invoice.pdf 1 "Update the date to January 31, 2026"
```

**Multiple changes:**
```bash
nano-pdf edit proposal.pdf 2 "Change the heading to 'Project Timeline', update the table to show Q1-Q4, and add a note at the bottom saying 'Subject to change'"
```

**Remove watermark:**
```bash
nano-pdf edit draft.pdf 1 "Remove the 'DRAFT' watermark"
```

## Best Practices

1. **Be specific** in instructions
2. **One page at a time** for best results
3. **Sanity-check output** before sending/sharing
4. **Test page numbering** if result is unexpected
5. **Backup original** for important documents

## Common Use Cases

**Update presentation:**
```bash
nano-pdf edit deck.pdf 1 "Update Q4 2025 to Q1 2026"
```

**Fix formatting:**
```bash
nano-pdf edit report.pdf 3 "Make all headings bold and increase font size to 14pt"
```

**Add footer:**
```bash
nano-pdf edit document.pdf 1 "Add footer with page number and copyright notice"
```

**Redact information:**
```bash
nano-pdf edit contract.pdf 2 "Black out the salary figure in section 3.2"
```

## Limitations

1. **AI-powered** - results may vary
2. **Page-by-page** - bulk edits need multiple commands
3. **Complex layouts** may be challenging
4. **Always verify** output before use

## Troubleshooting

**Wrong page edited:**
- Try other page numbering (0-based vs 1-based)

**Changes not applied:**
- Be more specific in instruction
- Break into smaller edits

**Output corrupted:**
- Try simpler instruction
- Check input PDF is valid

## Source
SKILL.md: ` \openclaw\skills\nano-pdf\SKILL.md`
