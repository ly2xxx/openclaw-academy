# 1Password CLI

**Location:** ` \openclaw\skills\1password\`  
**Emoji:** 🔐  
**Requirements:** `op` CLI, 1Password desktop app  
**Homepage:** https://developer.1password.com/docs/cli/get-started/

## Purpose
Set up and use 1Password CLI for reading/injecting/running secrets securely.

## Installation

```bash
brew install 1password-cli
```

## Setup Workflow

1. **Check CLI version:**
   ```bash
   op --version
   ```

2. **Enable desktop app integration** (in 1Password app settings)

3. **Unlock desktop app** (must be unlocked for CLI access)

4. **REQUIRED: Create tmux session** (no direct `op` calls!)

5. **Sign in inside tmux:**
   ```bash
   op signin
   ```

6. **Verify access:**
   ```bash
   op whoami
   ```

## ⚠️ REQUIRED: tmux Session (T-Max)

The shell tool uses fresh TTY per command. To avoid re-prompts:

**Always run `op` inside dedicated tmux session:**

```bash
SOCKET_DIR="${OPENCLAW_TMUX_SOCKET_DIR:-${TMPDIR:-/tmp}/openclaw-tmux-sockets}"
mkdir -p "$SOCKET_DIR"
SOCKET="$SOCKET_DIR/openclaw-op.sock"
SESSION="op-auth-$(date +%Y%m%d-%H%M%S)"

tmux -S "$SOCKET" new -d -s "$SESSION" -n shell
tmux -S "$SOCKET" send-keys -t "$SESSION":0.0 -- "op signin --account my.1password.com" Enter
tmux -S "$SOCKET" send-keys -t "$SESSION":0.0 -- "op whoami" Enter
tmux -S "$SOCKET" send-keys -t "$SESSION":0.0 -- "op vault list" Enter
tmux -S "$SOCKET" capture-pane -p -J -t "$SESSION":0.0 -S -200
tmux -S "$SOCKET" kill-session -t "$SESSION"
```

## Common Operations

### List & Search

```bash
# List vaults
op vault list

# List items
op item list

# Search items
op item list --vault Personal

# Get item details
op item get "Item Name"
```

### Read Secrets

```bash
# Get specific field
op read "op://vault/item/field"

# Get password
op read "op://Private/GitHub/password"
```

### Inject & Run

```bash
# Inject secrets into command
op run --env-file=.env -- ./script.sh

# Run with inline secrets
op run -- curl -u "op://vault/item/username:password" https://api.example.com
```

## Multiple Accounts

```bash
# Use specific account
op --account my.1password.com vault list

# Or set environment variable
export OP_ACCOUNT="my.1password.com"
```

## Guardrails

1. **Never paste secrets** into logs, chat, or code
2. **Prefer `op run`/`op inject`** over writing to disk
3. **Sign in without app integration:** use `op account add`
4. **"not signed in" error:** Re-run `op signin` in tmux
5. **Do NOT run `op` outside tmux** - stop and ask if unavailable

## References

- `references/get-started.md` - Install + app integration + sign-in
- `references/cli-examples.md` - Real `op` command examples

## Source
SKILL.md: ` \openclaw\skills\1password\SKILL.md`
