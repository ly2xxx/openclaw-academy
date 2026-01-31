# Himalaya - Email CLI

**Location:** ` \openclaw\skills\himalaya\`  
**Emoji:** 📧  
**Requirements:** `himalaya` CLI  
**Homepage:** https://github.com/pimalaya/himalaya

## Purpose
CLI email client managing emails via IMAP/SMTP from the terminal.

## Installation

```bash
brew install himalaya
```

## Setup

**Interactive wizard:**
```bash
himalaya account configure
```

**Manual config** (`~/.config/himalaya/config.toml`):
```toml
[accounts.personal]
email = "you@example.com"
display-name = "Your Name"
default = true

backend.type = "imap"
backend.host = "imap.example.com"
backend.port = 993
backend.encryption.type = "tls"
backend.login = "you@example.com"
backend.auth.type = "password"
backend.auth.cmd = "pass show email/imap"

message.send.backend.type = "smtp"
message.send.backend.host = "smtp.example.com"
message.send.backend.port = 587
message.send.backend.encryption.type = "start-tls"
message.send.backend.login = "you@example.com"
message.send.backend.auth.type = "password"
message.send.backend.auth.cmd = "pass show email/smtp"
```

## Common Operations

### List & Read

```bash
# List folders
himalaya folder list

# List inbox
himalaya envelope list

# List specific folder
himalaya envelope list --folder "Sent"

# Search
himalaya envelope list from john@example.com subject meeting

# Read email
himalaya message read 42
```

### Compose & Send

```bash
# Interactive compose (opens $EDITOR)
himalaya message write

# Quick send with headers
himalaya message write -H "To:recipient@example.com" -H "Subject:Test" "Body"

# Reply
himalaya message reply 42

# Reply all
himalaya message reply 42 --all

# Forward
himalaya message forward 42
```

### Organize

```bash
# Move to folder
himalaya message move 42 "Archive"

# Copy to folder
himalaya message copy 42 "Important"

# Delete
himalaya message delete 42

# Add/remove flags
himalaya flag add 42 --flag seen
himalaya flag remove 42 --flag seen
```

### Attachments

```bash
# Download attachments
himalaya attachment download 42

# Save to directory
himalaya attachment download 42 --dir ~/Downloads
```

## Multiple Accounts

```bash
# List accounts
himalaya account list

# Use specific account
himalaya --account work envelope list
```

## Output Formats

```bash
himalaya envelope list --output json
himalaya envelope list --output plain
```

## Debugging

```bash
# Debug logging
RUST_LOG=debug himalaya envelope list

# Full trace
RUST_LOG=trace RUST_BACKTRACE=1 himalaya envelope list
```

## Tips

1. **Use --help** for detailed command usage
2. **Message IDs** are relative to current folder
3. **MML syntax** for rich emails with attachments
4. **Secure passwords** using `pass`, keyring, or command

## References

- `references/configuration.md` - Config file setup
- `references/message-composition.md` - MML syntax for composing

## Source
SKILL.md: ` \openclaw\skills\himalaya\SKILL.md`
