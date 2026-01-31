# iMessage CLI (imsg)

**Location:** ` \openclaw\skills\imsg\`  
**Emoji:** 📨  
**Requirements:** `imsg` CLI, macOS only  
**Homepage:** https://imsg.to

## Purpose
Read and send iMessage/SMS via Messages.app CLI on macOS.

## Installation

```bash
brew install steipete/tap/imsg
```

## Requirements

1. **Messages.app signed in**
2. **Full Disk Access** for terminal
   - System Settings → Privacy & Security → Full Disk Access
3. **Automation permission** to control Messages.app (for sending)
   - System Settings → Privacy & Security → Automation

## Common Commands

### List Chats

```bash
imsg chats --limit 10 --json
```

### Read History

```bash
# Basic history
imsg history --chat-id 1 --limit 20

# With attachments
imsg history --chat-id 1 --limit 20 --attachments --json
```

### Watch Chat (Live)

```bash
# Monitor chat in real-time
imsg watch --chat-id 1 --attachments
```

### Send Message

```bash
# Text only
imsg send --to "+14155551212" --text "hi"

# With attachment
imsg send --to "+14155551212" --text "Check this out" --file /path/pic.jpg

# Service selection
imsg send --to "+14155551212" --text "hi" --service imessage
imsg send --to "+14155551212" --text "hi" --service sms
imsg send --to "+14155551212" --text "hi" --service auto  # Default
```

## Service Options

| Service | Description |
|---------|-------------|
| `imessage` | Force iMessage (blue bubble) |
| `sms` | Force SMS (green bubble) |
| `auto` | Let Messages.app decide (default) |

## Best Practices

1. **Confirm before sending** - Verify recipient and message
2. **Use --json** for scripting/parsing
3. **Check permissions** if commands fail
4. **Service auto** for reliability (handles fallback)
5. **Watch mode** for live monitoring

## Common Use Cases

**Check recent messages:**
```bash
imsg chats --limit 5 --json | jq '.[] | {name: .displayName, id: .chatId}'
```

**Read conversation:**
```bash
imsg history --chat-id 1 --limit 50 --json | jq '.[] | {from: .sender, text: .text}'
```

**Send notification:**
```bash
imsg send --to "+14155551212" --text "Task completed successfully"
```

**Monitor chat:**
```bash
imsg watch --chat-id 1 | while read line; do
  echo "New message: $line"
done
```

## Troubleshooting

**Permission errors:**
1. Grant Full Disk Access to Terminal
2. Grant Automation permission when prompted

**Service selection issues:**
- Use `--service auto` to let Messages decide
- Verify recipient is iMessage-capable for `--service imessage`

**macOS only:**
- Requires Messages.app (not available on other platforms)

## Source
SKILL.md: ` \openclaw\skills\imsg\SKILL.md`
