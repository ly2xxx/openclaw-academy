# GitHub Skill

**Location:** ` \openclaw\skills\github\`  
**Emoji:** 🐙  
**Requirements:** `gh` CLI (GitHub CLI)

## Purpose
Interact with GitHub repositories, PRs, issues, and CI/CD workflows using the `gh` command-line tool.

## Key Commands

### Pull Requests
- Check CI status: `gh pr checks <pr-number> --repo owner/repo`
- List workflow runs: `gh run list --repo owner/repo --limit 10`
- View run details: `gh run view <run-id> --repo owner/repo`
- View failed logs: `gh run view <run-id> --repo owner/repo --log-failed`

### API Access
- Advanced queries: `gh api repos/owner/repo/pulls/55 --jq '.title, .state'`
- JSON output: Most commands support `--json` flag
- JQ filtering: `--jq` for structured data extraction

## Installation
- **macOS:** `brew install gh`
- **Linux:** `apt install gh`

## Usage Pattern
Always specify `--repo owner/repo` when not in a git directory.

## Source
SKILL.md: ` \openclaw\skills\github\SKILL.md`
