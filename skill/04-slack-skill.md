# Slack Skill

**Location:** ` \openclaw\skills\slack\`  
**Emoji:** 💬  
**Requirements:** `channels.slack` configured in OpenClaw

## Purpose
Control Slack via the `slack` tool - reactions, pins, send/edit/delete messages, member info.

## Required Inputs

- `channelId` and `messageId` (Slack timestamp, e.g. `1712023032.1234`)
- For reactions: `emoji` (Unicode or `:name:`)
- For sends: `to` target (`channel:<id>` or `user:<id>`) and `content`

**Note:** Message context lines include `slack message id` and `channel` fields you can reuse directly.

## Action Groups

| Action Group | Default | Notes |
|--------------|---------|-------|
| reactions | enabled | React + list reactions |
| messages | enabled | Read/send/edit/delete |
| pins | enabled | Pin/unpin/list |
| memberInfo | enabled | Member info |
| emojiList | enabled | Custom emoji list |

## Key Actions

### React to Message
```json
{
  "action": "react",
  "channelId": "C123",
  "messageId": "1712023032.1234",
  "emoji": "✅"
}
```

### Send Message
```json
{
  "action": "sendMessage",
  "to": "channel:C123",
  "content": "Hello from OpenClaw"
}
```

### Edit Message
```json
{
  "action": "editMessage",
  "channelId": "C123",
  "messageId": "1712023032.1234",
  "content": "Updated text"
}
```

### Pin Message
```json
{
  "action": "pinMessage",
  "channelId": "C123",
  "messageId": "1712023032.1234"
}
```

### List Pinned Items
```json
{
  "action": "listPins",
  "channelId": "C123"
}
```

### Read Recent Messages
```json
{
  "action": "readMessages",
  "channelId": "C123",
  "limit": 20
}
```

### Member Info
```json
{
  "action": "memberInfo",
  "userId": "U123"
}
```

## Ideas

- React with ✅ to mark completed tasks
- Pin key decisions or weekly status updates

## Source
SKILL.md: ` \openclaw\skills\slack\SKILL.md`
