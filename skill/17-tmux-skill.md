# tmux - Terminal Multiplexer for Interactive CLIs

**Location:** ` \openclaw\skills\tmux\`  
**Emoji:** 🧵  
**Requirements:** `tmux` CLI (macOS/Linux)  
**OS Support:** macOS, Linux (use WSL on Windows)

## Purpose
Remote-control tmux sessions for interactive CLIs by sending keystrokes and scraping pane output.

**When to use:** Interactive TTY needed (coding agents, REPLs)  
**When NOT to use:** Long-running non-interactive tasks (use exec background mode)

## Quickstart (Isolated Socket)

```bash
SOCKET_DIR="${OPENCLAW_TMUX_SOCKET_DIR:-${TMPDIR:-/tmp}/openclaw-tmux-sockets}"
mkdir -p "$SOCKET_DIR"
SOCKET="$SOCKET_DIR/openclaw.sock"
SESSION=openclaw-python

tmux -S "$SOCKET" new -d -s "$SESSION" -n shell
tmux -S "$SOCKET" send-keys -t "$SESSION":0.0 -- 'PYTHON_BASIC_REPL=1 python3 -q' Enter
tmux -S "$SOCKET" capture-pane -p -J -t "$SESSION":0.0 -S -200
```

**Always print monitor commands after starting:**
```
To monitor:
  tmux -S "$SOCKET" attach -t "$SESSION"
  tmux -S "$SOCKET" capture-pane -p -J -t "$SESSION":0.0 -S -200
```

## Socket Convention

- Use `OPENCLAW_TMUX_SOCKET_DIR` (legacy `CLAWDBOT_TMUX_SOCKET_DIR` supported)
- Default socket: `"$OPENCLAW_TMUX_SOCKET_DIR/openclaw.sock"`

## Targeting & Naming

- **Target format:** `session:window.pane` (defaults to `:0.0`)
- **Keep names short** - avoid spaces
- **Inspect:**
  - `tmux -S "$SOCKET" list-sessions`
  - `tmux -S "$SOCKET" list-panes -a`

## Finding Sessions

```bash
# List sessions on socket
{baseDir}/scripts/find-sessions.sh -S "$SOCKET"

# Scan all sockets
{baseDir}/scripts/find-sessions.sh --all
```

## Sending Input Safely

**Literal sends (preferred):**
```bash
tmux -S "$SOCKET" send-keys -t target -l -- "$cmd"
```

**Control keys:**
```bash
tmux -S "$SOCKET" send-keys -t target C-c
```

## Watching Output

**Capture recent history:**
```bash
tmux -S "$SOCKET" capture-pane -p -J -t target -S -200
```

**Wait for prompts:**
```bash
{baseDir}/scripts/wait-for-text.sh -t session:0.0 -p 'pattern'
```

**Attach (detach with Ctrl+b d):**
```bash
tmux -S "$SOCKET" attach -t "$SESSION"
```

## Orchestrating Coding Agents

tmux excels at running **multiple coding agents in parallel**:

```bash
SOCKET="${TMPDIR:-/tmp}/codex-army.sock"

# Create multiple sessions
for i in 1 2 3 4 5; do
  tmux -S "$SOCKET" new-session -d -s "agent-$i"
done

# Launch agents in different workdirs
tmux -S "$SOCKET" send-keys -t agent-1 "cd /tmp/project1 && codex --yolo 'Fix bug X'" Enter
tmux -S "$SOCKET" send-keys -t agent-2 "cd /tmp/project2 && codex --yolo 'Fix bug Y'" Enter

# Poll for completion (check for prompt)
for sess in agent-1 agent-2; do
  if tmux -S "$SOCKET" capture-pane -p -t "$sess" -S -3 | grep -q "❯"; then
    echo "$sess: DONE"
  else
    echo "$sess: Running..."
  fi
done

# Get full output from completed session
tmux -S "$SOCKET" capture-pane -p -t agent-1 -S -500
```

### Tips for Parallel Agents

- Use separate **git worktrees** for parallel fixes (no branch conflicts)
- Run `pnpm install` first before codex in fresh clones
- Check for shell prompt (`❯` or `$`) to detect completion
- Codex needs `--yolo` or `--full-auto` for non-interactive fixes

## Spawning Processes

**For Python REPLs:**
```bash
PYTHON_BASIC_REPL=1 python3 -q
```

(Non-basic REPL breaks send-keys flows)

## Cleanup

**Kill session:**
```bash
tmux -S "$SOCKET" kill-session -t "$SESSION"
```

**Kill all sessions on socket:**
```bash
tmux -S "$SOCKET" list-sessions -F '#{session_name}' | xargs -r -n1 tmux -S "$SOCKET" kill-session -t
```

**Kill server:**
```bash
tmux -S "$SOCKET" kill-server
```

## Helper: wait-for-text.sh

Poll pane for regex/fixed string with timeout:

```bash
{baseDir}/scripts/wait-for-text.sh -t session:0.0 -p 'pattern' [-F] [-T 20] [-i 0.5] [-l 2000]
```

**Parameters:**
- `-t`/`--target` - Pane target (required)
- `-p`/`--pattern` - Regex to match (required); `-F` for fixed string
- `-T` - Timeout seconds (default 15)
- `-i` - Poll interval seconds (default 0.5)
- `-l` - History lines to search (default 1000)

## Windows Support

tmux supported on macOS/Linux. On Windows, use **WSL** and install tmux inside WSL.

## Source
SKILL.md: ` \openclaw\skills\tmux\SKILL.md`
