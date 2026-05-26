# WhatsApp Channel Architecture

**Directory:** `extensions/whatsapp/`  
**Language:** TypeScript  
**Plugin Type:** Channel Plugin  
**Protocol:** Baileys (WhatsApp Web Multi-Device)

## Purpose

The WhatsApp channel plugin integrates WhatsApp messaging with OpenClaw via:
- QR code authentication (Web WhatsApp)
- Message monitoring (inbound)
- Message sending (outbound)
- Group chat support
- Media attachments
- Reactions, polls

## Plugin Registration

**File:** `extensions/whatsapp/index.ts`

```typescript
const plugin = {
  id: "whatsapp",
  name: "WhatsApp",
  description: "WhatsApp channel plugin",
  configSchema: emptyPluginConfigSchema(),
  register(api: OpenClawPluginApi) {
    setWhatsAppRuntime(api.runtime);
    api.registerChannel({ plugin: whatsappPlugin });
  }
};
```

## Channel Plugin Interface

**File:** `extensions/whatsapp/src/channel.ts` (line ~1)

```typescript
export const whatsappPlugin: ChannelPlugin<ResolvedWhatsAppAccount> = {
  id: "whatsapp",
  meta: { ... },
  onboarding: whatsappOnboardingAdapter,
  agentTools: () => [createLoginTool()],
  pairing: { idLabel: "whatsappSenderId" },
  capabilities: {
    chatTypes: ["direct", "group"],
    polls: true,
    reactions: true,
    media: true
  },
  config: { ... },
  security: { ... },
  setup: { ... },
  groups: { ... },
  messaging: { ... },
  actions: { ... },
  status: { ... }
};
```

## Key Components

### 1. Authentication (QR Code Login)

**Flow:**
1. User runs `openclaw whatsapp login` or `web.login.start` gateway method
2. Gateway generates QR code
3. User scans with WhatsApp app
4. Baileys establishes authenticated session
5. Session stored in `authDir` (e.g., `~/.openclaw/whatsapp/auth-default/`)

**Gateway Methods:**
- `web.login.start` - Generate QR code, start login
- `web.login.wait` - Wait for scan completion

**Agent Tool:**
```typescript
createLoginTool() → Tool
```

Generates tool definition for agents to initiate QR login.

### 2. Account Configuration

**Config structure** (`~/.openclaw/openclaw.json`):
```json5
{
  "channels": {
    "whatsapp": {
      "accounts": {
        "default": {
          "enabled": true,
          "name": "Personal WhatsApp",
          "authDir": "~/.openclaw/whatsapp/auth-default/",
          "dmPolicy": "pairing",
          "allowFrom": ["+15551234567", "*"],
          "groupPolicy": "allowlist",
          "groups": {
            "1234567890-1589920128@g.us": {
              "name": "Family Group",
              "enabled": true,
              "requireMention": true
            }
          }
        }
      }
    }
  }
}
```

**Account resolution:**
```typescript
resolveWhatsAppAccount({ cfg, accountId }) → ResolvedWhatsAppAccount
```

**File:** `openclaw/plugin-sdk` exports

### 3. Security Policies

**DM Policy** (direct messages):
- `"pairing"` - Requires explicit pairing approval
- `"allowlist"` - Only allowed senders (from `allowFrom`)
- `"open"` - Anyone can message (dangerous!)

**Group Policy:**
- `"allowlist"` - Only configured groups
- `"open"` - Any group (with mention requirement)

**Pairing flow:**
```typescript
// User sends first message
// Gateway checks DmPolicy
if (dmPolicy === "pairing") {
  // Send pairing request to user (Master Yang)
  // Wait for approval
  // Add to allowFrom list
}
```

**File:** `src/channels/plugins/pairing.ts`

### 4. Message Monitoring

**Inbound message flow:**
```
WhatsApp → Baileys → Monitor →  Normalize → Gateway → Agent
```

**Message normalization:**
```typescript
normalizeWhatsAppMessagingTarget(target: string) → string
```

Converts WhatsApp JIDs to OpenClaw format:
- `15551234567@s.whatsapp.net` → `+15551234567`
- `1234567890-1589920128@g.us` → `whatsapp-group-1234567890-1589920128@g.us`

**File:** `openclaw/plugin-sdk`

### 5. Message Sending

**Messaging interface** (line ~280):
```typescript
messaging: {
  normalizeTarget: (target) => normalizeWhatsAppTarget(target),
  sendText: async ({ account, to, message, replyTo }) => { ... },
  sendMedia: async ({ account, to, mediaUrl, caption }) => { ... },
  sendReaction: async ({ account, messageId, emoji, remove }) => { ... },
  sendPoll: async ({ account, to, question, options, allowMultiselect }) => { ... }
}
```

**Target formats:**
- Direct: `+15551234567` or `15551234567@s.whatsapp.net`
- Group: `whatsapp-group-1234567890-1589920128@g.us`

**File:** `extensions/whatsapp/src/channel.ts` (line ~280)

### 6. Groups Support

**Group configuration:**
```typescript
groups: {
  listFromConfig: (cfg) => listWhatsAppDirectoryGroupsFromConfig(cfg),
  normalizeGroupId: (groupId) => normalizeWhatsAppTarget(groupId),
  isGroupJid: (jid) => isWhatsAppGroupJid(jid),
  resolveGroupToolPolicy: (cfg, accountId, groupId) => 
    resolveWhatsAppGroupToolPolicy(cfg, accountId, groupId),
  resolveMentionRequirement: (cfg, accountId, groupId) =>
    resolveWhatsAppGroupRequireMention(cfg, accountId, groupId)
}
```

**Mention detection:**
Groups require `@mention` to trigger agent:
```typescript
if (isGroupMessage && !mentioned) {
  return; // Ignore
}
```

**File:** `extensions/whatsapp/src/channel.ts` (line ~240)

### 7. Media Handling

**Supported media types:**
- Images (JPEG, PNG, WebP)
- Videos (MP4, 3GP)
- Audio (MP3, M4A, OGG)
- Documents (PDF, DOCX, etc.)
- Stickers

**Media sending:**
```typescript
sendMedia: async ({ account, to, mediaUrl, caption, mimeType }) => {
  // Download media from URL or read from file
  // Upload to WhatsApp servers
  // Send message with media reference
}
```

**Media limits:**
- Image: 16 MB
- Video: 16 MB (longer = automatic compression)
- Document: 100 MB
- Audio: 16 MB

**File:** `src/channels/plugins/media-limits.ts`

### 8. Actions Support

**Available actions:**
```typescript
actions: {
  poll: {
    execute: async ({ account, target, question, options, allowMultiselect }) => { ... }
  },
  react: {
    execute: async ({ account, target, messageId, emoji }) => { ... }
  },
  unreact: {
    execute: async ({ account, target, messageId }) => { ... }
  }
}
```

**Action gating:**
```typescript
const gate = createActionGate({
  config: account.actions ?? {},
  defaults: { polls: true, reactions: true }
});

if (!gate.isEnabled("polls")) {
  throw new Error("Polls action disabled");
}
```

**File:** `extensions/whatsapp/src/channel.ts` (line ~350)

### 9. Status & Health

**Status collection:**
```typescript
status: {
  collect: async ({ account, cfg }) => {
    const issues = collectWhatsAppStatusIssues({ account, cfg });
    return {
      connected: await isConnected(account),
      authenticated: await webAuthExists(account.authDir),
      issues
    };
  }
}
```

**Health checks:**
- Connection status
- Auth status (valid session)
- Group access warnings
- Configuration issues

**File:** `extensions/whatsapp/src/channel.ts` (line ~420)

## Baileys Integration

**Library:** `@whiskeysockets/baileys`  
**Protocol:** WhatsApp Web Multi-Device

**Key Baileys concepts:**
- **WASocket** - WebSocket connection to WhatsApp servers
- **AuthState** - Session credentials storage
- **Messages** - Inbound/outbound message objects
- **JID** - WhatsApp identifiers (phone@s.whatsapp.net, groupId@g.us)

**Connection initialization:**
```typescript
import makeWASocket from '@whiskeysockets/baileys';

const sock = makeWASocket({
  auth: authState,
  printQRInTerminal: false,
  logger: pino({ level: 'silent' })
});

sock.ev.on('connection.update', (update) => {
  // Handle connection state
});

sock.ev.on('messages.upsert', ({ messages }) => {
  // Handle incoming messages
});
```

## Directory Structure

```
extensions/whatsapp/
├── src/
│   ├── channel.ts       # Channel plugin definition
│   └── runtime.ts       # Runtime context
├── index.ts             # Plugin entry point
├── openclaw.plugin.json # Plugin metadata
└── package.json         # Dependencies (Baileys, etc.)
```

## Configuration Files

**Auth storage:** `~/.openclaw/whatsapp/auth-default/`
- `creds.json` - Session credentials
- `app-state-sync-key-*.json` - App state sync keys

**Session storage:** `~/.openclaw/agents/main/sessions/`
- `sessions.json` - Session index
- `whatsapp-<phone>.jsonl` - Message transcripts

## Message Flow

### Inbound

```
WhatsApp App
  ↓
WhatsApp Servers
  ↓
Baileys (WASocket)
  ↓
Message Event Handler
  ↓
Normalize Message
  ↓
Check Security (DM/Group policies)
  ↓
Gateway (server-channels.ts)
  ↓
Agent Runner
  ↓
Generate Reply
  ↓
Send via WhatsApp Channel
```

### Outbound

```
Agent Reply
  ↓
Gateway (normalizeTarget)
  ↓
WhatsApp Channel (sendText/sendMedia)
  ↓
Baileys Socket (sendMessage)
  ↓
WhatsApp Servers
  ↓
Recipient's WhatsApp App
```

## Heartbeat Integration

**Heartbeat recipients:**
```typescript
resolveWhatsAppHeartbeatRecipients(cfg) → string[]
```

Determines which WhatsApp contacts receive heartbeat messages (reminders, alerts).

**File:** `openclaw/plugin-sdk`

## Technology Stack

- **Protocol:** Baileys (WhatsApp Web)
- **Transport:** WebSocket (WSS)
- **Auth:** QR code scan
- **Storage:** JSON files (creds, state)
- **Media:** Direct upload to WhatsApp CDN
- **Language:** TypeScript
- **Dependencies:**
  - `@whiskeysockets/baileys` - WhatsApp client
  - `pino` - Logging
  - `sharp` - Image processing (optional)

## Important Files

| File | Purpose |
|------|---------|
| `extensions/whatsapp/index.ts` | Plugin registration |
| `extensions/whatsapp/src/channel.ts` | Channel implementation |
| `extensions/whatsapp/src/runtime.ts` | Runtime context |
| `src/channels/plugins/whatsapp-heartbeat.ts` | Heartbeat logic |
| `src/channels/plugins/catalog.ts` | Plugin registry |

## Known Limitations

1. **Single device:** One WhatsApp account per OpenClaw instance
2. **QR expiry:** QR codes expire after ~60 seconds (must re-generate)
3. **Rate limits:** WhatsApp may rate limit rapid messaging
4. **Group mentions:** Groups require `@mention` to trigger (configurable)
5. **Media size:** 16 MB limit for images/videos (automatic compression)

## Next Steps

- Explore other channels (Discord, Telegram)
- Review model providers (`src/providers/`)
- Understand cron scheduler (`src/cron/`)
- Study memory system (`src/memory/`)
