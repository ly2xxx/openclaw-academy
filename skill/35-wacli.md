# wacli - WhatsApp CLI (Third-Party Messaging)

**Location:** ` \openclaw\skills\wacli\`  
**Emoji:** 📱  
**Requirements:** `wacli` CLI  
**Homepage:** https://wacli.sh

## Purpose
Send WhatsApp messages to **other people** or search/sync WhatsApp history.

**⚠️ CRITICAL:** NOT for normal user chats! Only use when user explicitly asks to message someone else.

## Installation

```bash
brew install steipete/tap/wacli
```

**Or via Go:**
```bash
go install github.com/steipete/wacli/cmd/wacli@latest
```

## Safety Rules

1. **Require explicit recipient + message**
2. **Confirm before sending**
3. **If ambiguous, ask clarifying question**
4. **Don't use for normal chats** - OpenClaw routes automatically

## Auth & Sync

**QR login + initial sync:**
```bash
wacli auth
```

**Continuous sync:**
```bash
wacli sync --follow
```

**Diagnostics:**
```bash
wacli doctor
```

## Find Chats & Messages

### List Chats

```bash
wacli chats list --limit 20 --query "name or number"
```

### Search Messages

```bash
# Search in specific chat
wacli messages search "query" --limit 20 --chat <jid>

# Date range search
wacli messages search "invoice" --after 2025-01-01 --before 2025-12-31
```

## History Backfill

```bash
wacli history backfill --chat <jid> --requests 2 --count 50
```

**Note:** Requires phone online; best-effort results.

## Send Messages

### Text Message

```bash
wacli send text --to "+14155551212" --message "Hello! Are you free at 3pm?"
```

### Group Message

```bash
wacli send text --to "1234567890-123456789@g.us" --message "Running 5 min late."
```

### File with Caption

```bash
wacli send file --to "+14155551212" --file /path/agenda.pdf --caption "Agenda"
```

## JID Formats

**Direct chats:**
```
<number>@s.whatsapp.net
```

**Groups:**
```
<id>@g.us
```

**Find JIDs:**
```bash
wacli chats list --json | jq '.[] | {name, jid}'
```

## Configuration

**Store directory:** `~/.wacli` (override with `--store`)

**JSON output:** `--json` for machine-readable output

## Best Practices

1. **Use wacli ONLY for third-party messaging**
2. **Confirm recipient before sending**
3. **Use JIDs from `chats list`**
4. **Backfill requires phone online**
5. **Store in ~/.wacli by default**

## Common Use Cases

**Message colleague:**
```bash
wacli send text --to "+15551234567" --message "Meeting in 10 minutes"
```

**Send to group:**
```bash
# First, find group JID
wacli chats list --query "Family" --json

# Then send
wacli send text --to "1234567890-123456789@g.us" --message "Dinner at 7pm"
```

**Search history:**
```bash
wacli messages search "invoice" --after 2025-01-01
```

**Backfill conversation:**
```bash
wacli history backfill --chat "15551234567@s.whatsapp.net" --requests 5 --count 100
```

## Troubleshooting

**Authentication failed:**
- Run `wacli auth` to re-login via QR

**Phone offline:**
- Backfill requires phone to be online and connected

**Group not found:**
- Use `wacli chats list --json` to find group JID

## Source
SKILL.md: ` \openclaw\skills\wacli\SKILL.md`
