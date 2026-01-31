# Coding Agent - AI-Powered Development

**Location:** ` \openclaw\skills\coding-agent\`  
**Emoji:** 🧩  
**Requirements:** One of: `claude`, `codex`, `opencode`, `pi` CLI  
**⚠️ CRITICAL:** Always use `pty:true` (pseudo-terminal required!)

## Purpose
Run Codex CLI, Claude Code, OpenCode, or Pi Coding Agent programmatically for automated development tasks.

## ⚠️ PTY Mode Required

Coding agents are **interactive terminal applications** - they NEED a pseudo-terminal (PTY).

**Always use pty:true:**
```bash
# ✅ Correct
bash pty:true command:"codex exec 'Your prompt'"

# ❌ Wrong - agent will break!
bash command:"codex exec 'Your prompt'"
```

## Bash Tool Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `command` | string | Shell command to run |
| `pty` | boolean | **REQUIRED for agents!** Allocates pseudo-terminal |
| `workdir` | string | Working directory (agent sees only this folder) |
| `background` | boolean | Run in background, returns sessionId |
| `timeout` | number | Timeout in seconds |
| `elevated` | boolean | Run on host instead of sandbox |

## Process Tool Actions (Background Sessions)

| Action | Description |
|--------|-------------|
| `list` | List all running/recent sessions |
| `poll` | Check if session still running |
| `log` | Get session output (offset/limit) |
| `write` | Send raw data to stdin |
| `submit` | Send data + Enter |
| `send-keys` | Send key tokens or hex bytes |
| `paste` | Paste text (bracketed mode optional) |
| `kill` | Terminate session |

## Codex CLI

**Default model:** `gpt-5.2-codex` (set in ~/.codex/config.toml)

### Flags

| Flag | Effect |
|------|--------|
| `exec "prompt"` | One-shot execution, exits when done |
| `--full-auto` | Sandboxed but auto-approves in workspace |
| `--yolo` | NO sandbox, NO approvals (fastest, dangerous) |

### Quick Start

**One-shot (needs git repo):**
```bash
SCRATCH=$(mktemp -d) && cd $SCRATCH && git init && codex exec "Your prompt"
```

**In existing project:**
```bash
bash pty:true workdir:~/project command:"codex exec --full-auto 'Add error handling'"
```

### Background Mode

```bash
# Start agent
bash pty:true workdir:~/project background:true command:"codex --yolo 'Build snake game'"
# Returns sessionId

# Monitor
process action:log sessionId:XXX

# Check status
process action:poll sessionId:XXX

# Send input
process action:submit sessionId:XXX data:"yes"

# Kill if needed
process action:kill sessionId:XXX
```

### PR Reviews

**⚠️ CRITICAL: Never review in OpenClaw's own folder!**

**Safe approach - temp clone:**
```bash
REVIEW_DIR=$(mktemp -d)
git clone https://github.com/user/repo.git $REVIEW_DIR
cd $REVIEW_DIR && gh pr checkout 130
bash pty:true workdir:$REVIEW_DIR command:"codex review --base origin/main"
trash $REVIEW_DIR
```

**Alternative - git worktree:**
```bash
git worktree add /tmp/pr-130-review pr-130-branch
bash pty:true workdir:/tmp/pr-130-review command:"codex review --base main"
```

### Batch PR Reviews (Parallel Army!)

```bash
# Fetch all PR refs
git fetch origin '+refs/pull/*/head:refs/remotes/origin/pr/*'

# Deploy army - one Codex per PR
bash pty:true workdir:~/project background:true command:"codex exec 'Review PR #86. git diff origin/main...origin/pr/86'"
bash pty:true workdir:~/project background:true command:"codex exec 'Review PR #87. git diff origin/main...origin/pr/87'"

# Monitor all
process action:list

# Post results
gh pr comment <PR#> --body "<review content>"
```

## Claude Code

```bash
bash pty:true workdir:~/project command:"claude 'Your task'"
bash pty:true workdir:~/project background:true command:"claude 'Your task'"
```

## OpenCode

```bash
bash pty:true workdir:~/project command:"opencode run 'Your task'"
```

## Pi Coding Agent

```bash
# Install
npm install -g @mariozechner/pi-coding-agent

# Run
bash pty:true workdir:~/project command:"pi 'Your task'"

# Non-interactive
bash pty:true command:"pi -p 'Summarize src/'"

# Different provider
bash pty:true command:"pi --provider openai --model gpt-4o-mini -p 'Task'"
```

**Note:** Pi has Anthropic prompt caching enabled (Jan 2026)!

## Parallel Issue Fixing (Git Worktrees)

```bash
# Create worktrees
git worktree add -b fix/issue-78 /tmp/issue-78 main
git worktree add -b fix/issue-99 /tmp/issue-99 main

# Launch Codex in each
bash pty:true workdir:/tmp/issue-78 background:true command:"pnpm install && codex --yolo 'Fix #78. Commit & push.'"
bash pty:true workdir:/tmp/issue-99 background:true command:"pnpm install && codex --yolo 'Fix #99. Commit & push.'"

# Monitor
process action:list
process action:log sessionId:XXX

# Create PRs
cd /tmp/issue-78 && git push -u origin fix/issue-78
gh pr create --title "fix: ..." --body "..."

# Cleanup
git worktree remove /tmp/issue-78
git worktree remove /tmp/issue-99
```

## Auto-Notify on Completion

Add wake trigger to prompt:
```bash
bash pty:true workdir:~/project background:true command:"codex --yolo exec 'Build REST API.

When completely finished, run: openclaw gateway wake --text \"Done: Built todos API\" --mode now'"
```

Triggers immediate wake event instead of waiting for heartbeat!

## ⚠️ Critical Rules

1. **Always pty:true** - coding agents need terminal!
2. **Respect tool choice** - if user asks for Codex, use Codex
3. **Be patient** - don't kill slow sessions
4. **Monitor with process:log** - check progress without interfering
5. **--full-auto for building** - auto-approves changes
6. **vanilla for reviewing** - no special flags
7. **Parallel is OK** - run many processes for batch work
8. **NEVER in ~/clawd/** - will read soul docs!
9. **NEVER checkout branches in live OpenClaw repo**

## Progress Updates

Keep user in loop:
- Send 1 message when starting (what + where)
- Update only when:
  - Milestone completes
  - Agent asks question
  - Error occurs
  - Agent finishes (include what changed)
- If you kill session, immediately say why

## Learnings (Jan 2026)

- **PTY essential** - without it, output breaks or hangs
- **Git repo required** - Codex won't run outside git
- **exec is friend** - clean one-shots
- **submit vs write** - submit = input + Enter
- **Sass works** - Codex responds to playful prompts 🦞

## Source
SKILL.md: ` \openclaw\skills\coding-agent\SKILL.md`
