# Discord Skill

**Location:** ` \openclaw\skills\discord\`  
**Emoji:** 🎮  
**Requirements:** `channels.discord` configured in OpenClaw

## Purpose
Comprehensive Discord control: messages, reactions, stickers, polls, threads, pins, search, channels, moderation.

## Required Inputs

- **Reactions:** `channelId`, `messageId`, `emoji`
- **FetchMessage:** `guildId`, `channelId`, `messageId` OR `messageLink`  
  (e.g., `https://discord.com/channels/<guild>/<channel>/<message>`)
- **Send:** `to` target (`channel:<id>` or `user:<id>`), optional `content`
- **Polls:** `question` + 2-10 `answers`
- **Media:** `mediaUrl` (file:///path or https://...)
- **Emoji uploads:** `guildId`, `name`, `mediaUrl`, optional `roleIds` (≤256KB, PNG/JPG/GIF)
- **Sticker uploads:** `guildId`, `name`, `description`, `tags`, `mediaUrl` (≤512KB, PNG/APNG/Lottie JSON)

**Note:** `sendMessage` uses `to: "channel:<id>"` format, NOT `channelId`!

## Action Groups (All Enabled by Default Except Roles/Moderation)

| Group | Default | Actions |
|-------|---------|---------|
| reactions | enabled | react, reactions list, emojiList |
| messages | enabled | read/send/edit/delete |
| stickers | enabled | send stickers |
| polls | enabled | create polls |
| threads | enabled | create/list/reply |
| pins | enabled | pin/unpin/list |
| search | enabled | search messages |
| emojiUploads | enabled | upload custom emoji |
| stickerUploads | enabled | upload stickers |
| memberInfo | enabled | member details |
| roleInfo | enabled | role info |
| channelInfo | enabled | channel details |
| voiceStatus | enabled | voice status |
| events | enabled | scheduled events |
| roles | **disabled** | role add/remove |
| channels | **disabled** | create/edit/delete channels |
| moderation | **disabled** | timeout/kick/ban |

## Key Actions

### Reactions
```json
{"action": "react", "channelId": "123", "messageId": "456", "emoji": "✅"}
{"action": "reactions", "channelId": "123", "messageId": "456", "limit": 100}
```

### Send/Edit/Delete Messages
```json
{"action": "sendMessage", "to": "channel:123", "content": "Hello!"}
{"action": "sendMessage", "to": "channel:123", "content": "Audio!", "mediaUrl": "file:///tmp/audio.mp3"}
{"action": "editMessage", "channelId": "123", "messageId": "456", "content": "Fixed"}
{"action": "deleteMessage", "channelId": "123", "messageId": "456"}
```

### Stickers & Polls
```json
{"action": "sticker", "to": "channel:123", "stickerIds": ["9876543210"], "content": "Nice!"}
{"action": "poll", "to": "channel:123", "question": "Lunch?", "answers": ["Pizza", "Sushi"], "durationHours": 24}
```

### Threads
```json
{"action": "threadCreate", "channelId": "123", "name": "Bug triage", "messageId": "456"}
{"action": "threadList", "guildId": "999"}
{"action": "threadReply", "channelId": "777", "content": "Replying in thread"}
```

### Pins & Search
```json
{"action": "pinMessage", "channelId": "123", "messageId": "456"}
{"action": "listPins", "channelId": "123"}
{"action": "searchMessages", "guildId": "999", "content": "release notes", "limit": 10}
```

### Custom Emoji & Stickers
```json
{"action": "emojiUpload", "guildId": "999", "name": "party_blob", "mediaUrl": "file:///tmp/party.png"}
{"action": "stickerUpload", "guildId": "999", "name": "openclaw_wave", "description": "Waving", "tags": "👋", "mediaUrl": "file:///tmp/wave.png"}
```

### Channel Management (disabled by default)
```json
{"action": "channelCreate", "guildId": "999", "name": "general-chat", "type": 0, "parentId": "888"}
{"action": "categoryCreate", "guildId": "999", "name": "Projects"}
{"action": "channelEdit", "channelId": "123", "name": "new-name", "topic": "Updated"}
{"action": "channelDelete", "channelId": "123"}
```

### Moderation (disabled by default)
```json
{"action": "timeout", "guildId": "999", "userId": "111", "durationMinutes": 10}
```

## Discord Writing Style Guide 🎮

**Keep it conversational!** Discord is a chat platform, not documentation.

### ✅ Do
- Short, punchy messages (1-3 sentences)
- Multiple quick replies > one wall of text
- Use emoji for tone 🦞
- Lowercase casual style is fine
- Match the conversation energy

### ❌ Don't
- No markdown tables (renders as ugly raw `| text |`)
- No `## Headers` for casual chat (use **bold** or CAPS)
- Avoid multi-paragraph essays
- Skip "I'd be happy to help!" fluff

### Formatting That Works
- **bold** for emphasis
- `code` for technical terms
- Lists for multiple items
- > quotes for referencing
- Wrap multiple links in `<>` to suppress embeds

## Ideas
- React with ✅/⚠️ for status tracking
- Quick polls for decisions
- Celebratory stickers after deploys
- Weekly priority check polls

## Source
SKILL.md: ` \openclaw\skills\discord\SKILL.md`
