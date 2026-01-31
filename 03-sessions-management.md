# Sessions Management Architecture

**Directory:** `src/config/sessions/`, `src/gateway/session-utils.ts`  
**Language:** TypeScript  
**Storage:** JSON5 + JSONL files

## Purpose

Sessions track conversation state across different messaging platforms (WhatsApp, Discord, Telegram, etc.).

Each session has:
- Unique session key (agent:scope:session-id)
- Message transcript (JSONL file)
- Delivery context (channel, user, thread)
- Metadata (title, labels, model overrides)

## Session Key Format

```
agent:scope:session-id
```

**Examples:**
- `agent:main:main` - Main direct chat with user
- `agent:main:discord-chan-123` - Discord channel session
- `agent:main:whatsapp-group-456` - WhatsApp group session

**Parsing:**
```typescript
parseAgentSessionKey(key: string) → {
  agentId: string;
  scope: string;
  sessionId: string;
}
```

**File:** `src/routing/session-key.js`

## Storage Structure

```
~/.openclaw/agents/<agentId>/
  ├─ sessions.json          # Session index
  └─ sessions/
      ├─ <session-id>.jsonl # Message transcript
      └─ <session-id>.json  # Session metadata (optional)
```

### sessions.json (Index)

**File:** `~/.openclaw/agents/<agentId>/sessions.json`  
**Format:** JSON5

```json5
{
  "agent:main:main": {
    "sessionId": "main",
    "channel": "whatsapp",
    "to": "+15551234567",
    "deliveryContext": {
      "channel": "whatsapp",
      "to": "+15551234567",
      "accountId": "..."
    },
    "updatedAt": 1706713200000,
    "createdAt": 1706713200000
  }
}
```

**Key fields:**
- `sessionId` - Unique session identifier
- `channel` - Messaging platform (whatsapp, discord, telegram)
- `to` - Recipient identifier (phone number, channel ID, etc.)
- `deliveryContext` - Full delivery routing info
- `updatedAt` - Last message timestamp
- `createdAt` - Session creation timestamp

### Session Transcript (JSONL)

**File:** `~/.openclaw/agents/<agentId>/sessions/<session-id>.jsonl`  
**Format:** JSONL (JSON Lines - one JSON object per line)

Each line is either:

**Session metadata line:**
```json
{
  "type": "session",
  "timestamp": "2026-01-31T14:00:00.000Z",
  "sessionId": "main"
}
```

**Message line:**
```json
{
  "type": "message",
  "timestamp": "2026-01-31T14:00:01.000Z",
  "message": {
    "role": "user",
    "content": [
      { "type": "text", "text": "Hello!" }
    ]
  }
}
```

**Assistant message:**
```json
{
  "type": "message",
  "timestamp": "2026-01-31T14:00:02.000Z",
  "message": {
    "role": "assistant",
    "content": [
      { "type": "text", "text": "Hi! How can I help?" }
    ]
  },
  "usage": {
    "inputTokens": 100,
    "outputTokens": 50,
    "cost": { "total": 0.001 }
  }
}
```

## Session Store Management

### Load Session Store

```typescript
loadSessionStore(storePath: string, opts?) → Record<string, SessionEntry>
```

**Location:** `src/config/sessions/store.ts` (line ~94)

**Caching:**
- **TTL:** 45 seconds (configurable via `OPENCLAW_SESSION_CACHE_TTL_MS`)
- **Cache key:** `storePath`
- **Invalidation:** File mtime change
- **Deep copy:** Returns cloned store to prevent mutations

**Flow:**
1. Check cache (if enabled)
2. Validate cache (TTL + mtime)
3. Load from disk if cache miss
4. Normalize delivery fields
5. Update cache
6. Return store

### Save Session Store

```typescript
saveSessionStore(storePath: string, store: Record<string, SessionEntry>)
```

**Location:** `src/config/sessions/store.ts` (line ~177)

**Flow:**
1. Normalize delivery context
2. Sort entries by updatedAt
3. Write to temp file
4. Atomic rename to target
5. Invalidate cache

### Update Session Entry

```typescript
updateSessionEntry(
  storePath: string,
  sessionKey: string,
  patch: Partial<SessionEntry>
) → SessionEntry
```

**Location:** `src/config/sessions/store.ts` (line ~202)

**Flow:**
1. Load store
2. Merge patch with existing entry
3. Update `updatedAt` timestamp
4. Save store
5. Return updated entry

## Session Types

### Main Session

**Key:** `agent:main:main`

Direct chat with user across all channels. Uses routing to determine which channel to reply on.

```typescript
resolveMainSessionKey(agentId: string) → string
```

**File:** `src/config/sessions/main-session.ts`

### Group Sessions

**Pattern:** `agent:main:<channel>-<group-id>`

**Examples:**
- `agent:main:discord-chan-123456789`
- `agent:main:whatsapp-group-1234567890-1589920128@g.us`

```typescript
buildGroupDisplayName(channel: string, groupId: string) → string
```

**File:** `src/config/sessions/group.ts`

### Wizard Sessions

**Pattern:** `wizard-<uuid>`

Temporary sessions for onboarding/setup wizards.

**File:** `src/gateway/server-wizard-sessions.ts`

## Delivery Context

**Purpose:** Track where to send replies

```typescript
type DeliveryContext = {
  channel: string;           // e.g., "whatsapp"
  to: string;                // e.g., "+15551234567"
  accountId?: string;        // Channel account ID
  threadId?: string;         // Thread/channel ID
  replyToMessageId?: string; // Reply to specific message
};
```

**Normalization:**
```typescript
normalizeDeliveryContext(context: DeliveryContext) → DeliveryContext
```

Ensures consistent field naming and removes nulls.

**File:** `src/utils/delivery-context.ts`

### Delivery Context Resolution

**Priority order:**
1. Explicit `deliveryContext` in session entry
2. Fallback to `lastChannel`, `lastTo`, etc.
3. Infer from message context

**Function:**
```typescript
deliveryContextFromSession(entry: SessionEntry, msgContext?: MsgContext) → DeliveryContext
```

**File:** `src/utils/delivery-context.ts`

## Session Metadata

### Title Derivation

**Automatic title generation:**
```typescript
deriveTitleFromFirstMessage(text: string, maxLen: number) → string
```

**Rules:**
- Truncate at `DERIVED_TITLE_MAX_LEN` (60 chars)
- Remove code blocks, fences
- Strip excessive whitespace
- Smart truncation (prefer word boundaries)

**File:** `src/config/sessions/metadata.ts`

### Labels

Custom tags for session organization:

```typescript
type SessionEntry = {
  labels?: string[];  // e.g., ["work", "project-x"]
}
```

### Model Overrides

**Per-session model override:**
```typescript
type SessionEntry = {
  model?: string;  // e.g., "anthropic/claude-opus-4-5"
}
```

**File:** `src/sessions/model-overrides.ts`

## Session Transcript Operations

### Read Messages

```typescript
readSessionMessages(
  transcriptPath: string,
  limit?: number,
  offset?: number
) → Message[]
```

**File:** `src/gateway/session-utils.fs.ts`

### Read Preview

```typescript
readLastMessagePreviewFromTranscript(transcriptPath: string) → string | null
```

Returns last message text for session list preview.

**File:** `src/gateway/session-utils.fs.ts`

### Append Message

```typescript
appendToSessionTranscript(
  transcriptPath: string,
  message: Message
)
```

Appends JSONL line to transcript file.

## Session List API

```typescript
listSessions(opts: {
  agentId?: string;
  scope?: string;
  limit?: number;
  filters?: {
    kinds?: string[];
    activeMinutes?: number;
  };
}) → SessionsListResult
```

**Returns:**
```typescript
type SessionsListResult = {
  sessions: GatewaySessionRow[];
  defaults: GatewaySessionsDefaults;
}

type GatewaySessionRow = {
  sessionKey: string;
  sessionId: string;
  agentId: string;
  scope: string;
  title?: string;
  labels?: string[];
  channel?: string;
  to?: string;
  updatedAt?: number;
  createdAt?: number;
  preview?: string;
}
```

**File:** `src/gateway/session-utils.ts` (line ~700)

## Cache Management

### Session Store Cache

**Key:** `storePath`  
**TTL:** 45 seconds (default)  
**Validation:** File mtime comparison

```typescript
clearSessionStoreCacheForTest() // Test only
invalidateSessionStoreCache(storePath: string)
```

### Cache Config

**Environment variable:**
```bash
OPENCLAW_SESSION_CACHE_TTL_MS=45000  # 45 seconds
```

**Disable caching:**
```bash
OPENCLAW_SESSION_CACHE_TTL_MS=0
```

## Technology Stack

- **Storage Format:** JSON5 (index), JSONL (transcripts)
- **File I/O:** Node.js `fs` module (synchronous)
- **Caching:** In-memory Map with TTL
- **Serialization:** `JSON5.parse()`, `JSON5.stringify()`
- **Atomicity:** Temp file + rename for updates

## Important Files

| File | Purpose |
|------|---------|
| `config/sessions/store.ts` | Session store CRUD |
| `config/sessions/metadata.ts` | Title/label management |
| `config/sessions/main-session.ts` | Main session logic |
| `config/sessions/group.ts` | Group session logic |
| `config/sessions/types.ts` | Session types |
| `gateway/session-utils.ts` | Gateway session API |
| `gateway/session-utils.fs.ts` | Transcript file operations |
| `utils/delivery-context.ts` | Delivery routing |

## Next Steps

- Explore channel implementations (`src/channels/`)
- Review model providers (`src/providers/`)
- Understand cron scheduler (`src/cron/`)
