# BlueBubbles Plugin - iMessage Integration

**Location:** ` \openclaw\skills\bluebubbles\`  
**Purpose:** Build/update the BlueBubbles external channel plugin for OpenClaw

## Overview

BlueBubbles enables iMessage integration with OpenClaw through:
- REST API for sending messages
- Webhook handling for receiving messages
- Reaction/typing/read receipt support

## Layout

| Component | Location | Purpose |
|-----------|----------|---------|
| **Extension package** | `extensions/bluebubbles/` | Entry: `index.ts` |
| **Channel impl** | `src/channel.ts` | Main channel logic |
| **Webhook handler** | `src/monitor.ts` | Inbound message processing |
| **REST helpers** | `src/send.ts`, `src/probe.ts` | Outbound operations |
| **Runtime bridge** | `src/runtime.ts` | Set via `api.runtime` |
| **Catalog entry** | `src/channels/plugins/catalog.ts` | Onboarding |

## Internal Helpers (Use These!)

**Health & Delivery:**
- `probeBlueBubbles()` - Health checks
- `sendMessageBlueBubbles()` - Text delivery
- `resolveChatGuidForTarget()` - Chat lookup

**Reactions & Interactions:**
- `sendBlueBubblesReaction()` - Tapbacks (❤️, 👍, etc.)
- `sendBlueBubblesTyping()` - Typing indicators
- `markBlueBubblesChatRead()` - Read receipts

**Media:**
- `downloadBlueBubblesAttachment()` - Inbound media

**Shared Plumbing:**
- `buildBlueBubblesApiUrl()` - URL construction
- `blueBubblesFetchWithTimeout()` - REST requests

## Webhooks

BlueBubbles posts JSON to gateway HTTP server:

1. **Normalize** sender/chat IDs defensively (payloads vary by version)
2. **Skip** messages marked as from self
3. **Route** into core reply pipeline via `api.runtime`
4. **Attachments/stickers:** Use `<media:...>` placeholders when text empty
5. **Media paths:** Attach via `MediaUrl(s)` in inbound context

## Configuration

**Core settings** (`~/.openclaw/openclaw.json`):
```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://...",
      "password": "...",
      "webhookPath": "/webhook/bluebubbles",
      "actions": {
        "reactions": true
      }
    }
  }
}
```

## Message Tool Notes

**Reactions:**
```json
{
  "action": "react",
  "target": "+15551234567",
  "messageId": "ABC123",
  "emoji": "❤️"
}
```

**Note:** Requires both `target` (phone/chat ID) AND `messageId`

## Development Workflow

1. **Clone/navigate** to `extensions/bluebubbles/`
2. **Install deps:** `pnpm install`
3. **Build:** `pnpm build`
4. **Test webhook:** POST to configured webhook path
5. **Check logs** for normalization/routing issues

## Best Practices

1. **Use internal helpers** - Don't call BlueBubbles API directly
2. **Defensive normalization** - Payloads vary by BB version
3. **Skip self-messages** - Check sender before processing
4. **Media placeholders** - Use `<media:...>` format
5. **Error handling** - BB server may be down/unreachable

## Source
SKILL.md: ` \openclaw\skills\bluebubbles\SKILL.md`
