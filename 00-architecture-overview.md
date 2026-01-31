# OpenClaw Architecture Overview

**Last Updated:** 2026-01-31  
**Repository:** https://github.com/openclaw/openclaw  
**Language:** TypeScript (Node.js ≥22)

## System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Gateway Server                        │
│               (WebSocket Control Plane)                  │
│                   Port: 18789                            │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │   Agents     │  │   Sessions   │  │   Channels   │ │
│  │              │  │              │  │              │ │
│  │ - Pi Runner  │  │ - JSONL      │  │ - WhatsApp   │ │
│  │ - Tools      │  │ - Index      │  │ - Discord    │ │
│  │ - Sandbox    │  │ - Delivery   │  │ - Telegram   │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │   Providers  │  │     Cron     │  │    Memory    │ │
│  │              │  │              │  │              │ │
│  │ - Anthropic  │  │ - Heartbeat  │  │ - Search     │ │
│  │ - Google     │  │ - Wake       │  │ - MEMORY.md  │ │
│  │ - OpenAI     │  │ - Isolated   │  │ - Daily logs │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │    Nodes     │  │   Plugins    │  │   Canvas     │ │
│  │              │  │              │  │              │ │
│  │ - Mac        │  │ - Registry   │  │ - HTTP:18793 │ │
│  │ - iOS        │  │ - Extensions │  │ - WebView    │ │
│  │ - Android    │  │ - Skills     │  │ - LiveReload │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## Core Components

### 1. Gateway Server
**File:** `src/gateway/server.impl.ts`  
**Port:** 18789 (default)  
**Purpose:** Central control plane

**Responsibilities:**
- WebSocket connection management
- HTTP endpoints (Control UI, OpenAI API)
- Client/node coordination
- Plugin lifecycle
- Configuration hot-reload

**Key functions:**
- `startGatewayServer(port, opts)` → `GatewayServer` (line ~137)
- `createGatewayRuntimeState()` → Runtime state
- `createChannelManager()` → Channel lifecycle

**Documentation:** [01-gateway-overview.md](01-gateway-overview.md)

### 2. Agent Runtime
**Directory:** `src/agents/`  
**Core:** `pi-embedded-runner/run.ts`  
**Purpose:** Execute AI model requests

**Responsibilities:**
- Model selection & failover
- Auth profile rotation
- Context window management
- Tool execution
- Session compaction

**Key functions:**
- `runEmbeddedPiAgent(params)` → `EmbeddedPiRunResult` (line ~74)
- `resolveModel(provider, modelId, agentDir, config)`
- `runEmbeddedAttempt(attemptParams)` → Attempt result

**Documentation:** [02-agent-runtime.md](02-agent-runtime.md)

### 3. Sessions Management
**Directory:** `src/config/sessions/`, `src/gateway/session-utils.ts`  
**Storage:** JSON5 (index) + JSONL (transcripts)  
**Purpose:** Track conversation state

**Responsibilities:**
- Session key resolution (agent:scope:session-id)
- Message transcript storage
- Delivery context routing
- Metadata (title, labels, model overrides)

**Key functions:**
- `loadSessionStore(storePath)` → Session index
- `updateSessionEntry(storePath, sessionKey, patch)`
- `readSessionMessages(transcriptPath, limit, offset)`

**Storage:**
- `~/.openclaw/agents/<agentId>/sessions.json` - Index
- `~/.openclaw/agents/<agentId>/sessions/<session-id>.jsonl` - Transcript

**Documentation:** [03-sessions-management.md](03-sessions-management.md)

### 4. Channels (WhatsApp)
**Directory:** `extensions/whatsapp/`  
**Protocol:** Baileys (WhatsApp Web Multi-Device)  
**Purpose:** Messaging platform integration

**Responsibilities:**
- QR code authentication
- Message monitoring (inbound)
- Message sending (outbound)
- Group chat support
- Media attachments
- Security policies (DM/Group)

**Key components:**
- `whatsappPlugin` - Channel plugin definition
- `normalizeWhatsAppTarget()` - Target normalization
- `sendText()`, `sendMedia()`, `sendReaction()`, `sendPoll()`

**Documentation:** [04-channels-whatsapp.md](04-channels-whatsapp.md)

### 5. Cron Scheduler
**Directory:** `src/cron/`  
**Storage:** `~/.openclaw/cron/jobs.json`  
**Purpose:** Scheduled tasks & reminders

**Responsibilities:**
- Heartbeat polls (periodic checks)
- Wake events (immediate triggers)
- Isolated agent jobs
- Run history logging

**Key classes:**
- `CronService` - Main scheduler
- `buildGatewayCronService()` - Gateway integration

**Job types:**
- **Heartbeat:** Periodic checks (HEARTBEAT.md context)
- **Wake:** Immediate notifications
- **Isolated:** One-shot agent runs (separate session)

**Documentation:** [05-cron-scheduler.md](05-cron-scheduler.md)

### 6. Model Providers
**Directory:** `src/providers/`  
**Supported:** Anthropic, Google, OpenAI, xAI, GitHub Copilot

**Key files:**
- `github-copilot-auth.ts` - GitHub Copilot token exchange
- `github-copilot-models.ts` - Model catalog
- `qwen-portal-oauth.ts` - Qwen OAuth flow

**Model catalog:** `src/agents/model-catalog.ts`

### 7. Memory System
**Directory:** `src/memory/`, `src/agents/memory-search.ts`  
**Storage:** `MEMORY.md`, `memory/YYYY-MM-DD.md`  
**Purpose:** Long-term context & recall

**Key functions:**
- `memory_search(query)` - Semantic search across memory files
- `memory_get(path, from, lines)` - Snippet retrieval

### 8. Node Registry
**File:** `src/gateway/node-registry.ts`  
**Purpose:** Track paired devices (Mac, iOS, Android)

**Capabilities:**
- Camera snapshots
- Screen recording
- Location access
- Remote command execution

### 9. Plugin System
**Directory:** `src/plugins/`  
**Types:** Channel plugins, Extension plugins, Skills

**Plugin registry:**
- `requireActivePluginRegistry()` → Registry instance
- `loadGatewayPlugins()` → Loaded plugins

**Plugin locations:**
- `extensions/` - Built-in plugins (WhatsApp, Discord, etc.)
- `~/.openclaw/plugins/` - User-installed plugins
- `skills/` - Agent skills

### 10. Canvas Host
**File:** `src/canvas-host/server.ts`  
**Port:** 18793 (default)  
**Purpose:** Serve HTML/CSS/JS to paired nodes

**Features:**
- Static file serving
- Live reload (WebSocket)
- Tailscale integration

**Root:** `~/clawd/canvas/`  
**URL:** `http://<hostname>:18793/__moltbot__/canvas/<file>.html`

## Data Flow

### Inbound Message Flow
```
WhatsApp/Discord/etc
  ↓
Channel Plugin (monitor)
  ↓
Normalize Message
  ↓
Security Check (DM/Group policies)
  ↓
Gateway (server-channels.ts)
  ↓
Session Resolution
  ↓
Agent Runner (pi-embedded-runner)
  ↓
Model Provider (Anthropic/Google/OpenAI)
  ↓
Tool Execution (if needed)
  ↓
Generate Response
  ↓
Send via Channel
```

### Outbound Reply Flow
```
Agent Response
  ↓
Gateway (normalizeTarget)
  ↓
Channel Plugin (sendText/sendMedia)
  ↓
Protocol Layer (Baileys/Discord.js/etc)
  ↓
Platform Servers
  ↓
Recipient
```

## Directory Structure

```
openclaw/
├── src/
│   ├── gateway/          # Gateway server
│   ├── agents/           # Agent runtime
│   ├── sessions/         # Session types
│   ├── config/           # Config & sessions
│   ├── channels/         # Channel dock
│   ├── providers/        # Model providers
│   ├── cron/             # Cron scheduler
│   ├── memory/           # Memory search
│   ├── plugins/          # Plugin runtime
│   ├── canvas-host/      # Canvas server
│   └── infra/            # Infrastructure
│
├── extensions/
│   ├── whatsapp/         # WhatsApp plugin
│   ├── discord/          # Discord plugin
│   ├── telegram/         # Telegram plugin
│   └── ...               # Other channels
│
├── packages/
│   ├── clawdbot/         # Main package
│   └── moltbot/          # Kanban board
│
├── skills/               # Bundled skills
├── ui/                   # Control UI
└── apps/                 # macOS/iOS/Android apps
```

## Configuration

**Main config:** `~/.openclaw/openclaw.json`  
**Format:** JSON5

```json5
{
  "gateway": {
    "port": 18789,
    "bind": "auto",  // loopback|lan|tailnet|auto
    "controlUi": { "enabled": true }
  },
  "agents": {
    "defaults": {
      "model": {
        "provider": "anthropic",
        "id": "claude-sonnet-4-5",
        "fallbacks": [
          { "provider": "anthropic", "id": "claude-opus-4-5" },
          { "provider": "google", "id": "gemini-2.5-pro" }
        ]
      }
    }
  },
  "channels": {
    "whatsapp": {
      "accounts": {
        "default": {
          "enabled": true,
          "authDir": "~/.openclaw/whatsapp/auth-default/",
          "dmPolicy": "pairing",
          "allowFrom": ["*"]
        }
      }
    }
  },
  "cron": {
    "enabled": true,
    "store": "~/.openclaw/cron"
  }
}
```

## Technology Stack

**Core:**
- **Runtime:** Node.js ≥22
- **Language:** TypeScript
- **Package Manager:** pnpm (preferred), npm, or bun
- **Monorepo:** Multiple packages (clawdbot, moltbot)

**Networking:**
- **WebSocket:** `ws` library
- **HTTP:** Node.js `http`/`https`
- **Discovery:** mDNS/Bonjour (`bonjour-service`)
- **Tailscale:** Network mesh (optional)

**Messaging:**
- **WhatsApp:** `@whiskeysockets/baileys`
- **Discord:** `discord.js`
- **Telegram:** `node-telegram-bot-api`

**AI/ML:**
- **Anthropic:** `@anthropic-ai/sdk`
- **Google:** `@google/generative-ai`
- **OpenAI:** `openai` SDK

**Storage:**
- **Config:** JSON5 format
- **Sessions:** JSONL (JSON Lines)
- **Memory:** Markdown files
- **Cron:** JSON (jobs + logs)

## Important Patterns

### 1. Session Keys
**Format:** `agent:scope:session-id`

Examples:
- `agent:main:main` - Main direct chat
- `agent:main:discord-chan-123` - Discord channel
- `agent:main:whatsapp-group-456` - WhatsApp group

### 2. Delivery Context
```typescript
type DeliveryContext = {
  channel: string;      // "whatsapp", "discord", etc.
  to: string;           // Recipient ID
  accountId?: string;   // Channel account
  threadId?: string;    // Thread/channel ID
}
```

### 3. Model Failover
1. Try primary model (Claude Sonnet)
2. Auth error → Try next auth profile
3. Rate limit → Cooldown + next profile
4. Context overflow → Compact session + retry
5. All profiles fail → Try fallback model (Opus → Gemini)

### 4. Tool Execution
```typescript
// Tools return results that agents can read
{
  "tool": "bash",
  "action": "exec",
  "command": "ls -la"
}
```

Tool results are:
- Inserted into conversation
- Sanitized for display
- Logged to session transcript

## Documentation Index

| File | Component |
|------|-----------|
| [00-architecture-overview.md](00-architecture-overview.md) | **This file** - System overview |
| [01-gateway-overview.md](01-gateway-overview.md) | Gateway server architecture |
| [02-agent-runtime.md](02-agent-runtime.md) | Agent execution & failover |
| [03-sessions-management.md](03-sessions-management.md) | Session storage & routing |
| [04-channels-whatsapp.md](04-channels-whatsapp.md) | WhatsApp channel plugin |
| [05-cron-scheduler.md](05-cron-scheduler.md) | Cron jobs & heartbeats |

## Key Source Files

| File | Lines | Purpose |
|------|-------|---------|
| `src/gateway/server.impl.ts` | ~700 | Gateway initialization |
| `src/agents/pi-embedded-runner/run.ts` | ~700 | Agent execution |
| `src/config/sessions/store.ts` | ~470 | Session storage |
| `extensions/whatsapp/src/channel.ts` | ~500 | WhatsApp integration |
| `src/cron/service.ts` | ~300 | Cron scheduler |

## Development Commands

```bash
# Start gateway
openclaw gateway start

# Hot reload config
openclaw config reload

# List sessions
openclaw sessions list

# Cron management
openclaw cron list
openclaw cron add --text "..." --schedule "0 9 * * *"

# Channel login
openclaw whatsapp login

# Health check
openclaw doctor
```

## Next Steps for Deep Dive

1. **Agent Tools** - Explore `src/agents/tools/` for tool implementations
2. **Skills System** - Review `src/agents/skills/` for skill loading
3. **Molt Integration** - Study `packages/moltbot/` for Kanban board
4. **Control UI** - Check `ui/` for web interface
5. **Mobile Apps** - Investigate `apps/` for iOS/Android implementations

## Resources

- **Docs:** `{user_directory}\AppData\Roaming\npm\node_modules\clawdbot\docs`
- **Source:** https://github.com/openclaw/openclaw
- **Community:** https://discord.com/invite/clawd
- **Skills:** https://clawdhub.com
