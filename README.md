# OpenClaw Architecture Documentation

> **⚠️ OpenClaw Academy - Unofficial Community Documentation**
>
> This documentation was created by independent research and is **not affiliated with or endorsed by** the OpenClaw project.
>
> **Official source:** https://github.com/openclaw/openclaw
>
> This documentation is provided under the same license as OpenClaw for educational purposes.

---

**Created:** 2026-01-31
**Purpose:** Detailed technical architecture documentation for OpenClaw

This directory contains deep-dive architecture documentation extracted from the OpenClaw source code (` \openclaw`).

## 📚 Documentation Index

### Overview
**[00-architecture-overview.md](00-architecture-overview.md)** - System-wide architecture  
Complete system overview with component relationships, data flows, and technology stack.

### Core Components

1. **[01-gateway-overview.md](01-gateway-overview.md)** - Gateway Server  
   WebSocket control plane, HTTP endpoints, plugin management
   
2. **[02-agent-runtime.md](02-agent-runtime.md)** - Agent Execution  
   Model selection, auth profiles, failover logic, tool execution
   
3. **[03-sessions-management.md](03-sessions-management.md)** - Session Storage  
   Conversation state, JSONL transcripts, delivery routing
   
4. **[04-channels-whatsapp.md](04-channels-whatsapp.md)** - WhatsApp Channel  
   Baileys integration, QR auth, message monitoring, security policies
   
5. **[05-cron-scheduler.md](05-cron-scheduler.md)** - Cron Service  
   Scheduled jobs, heartbeat polls, wake events, isolated runs

## 🗺️ Architecture at a Glance

```
OpenClaw System
├── Gateway (Port 18789)
│   ├── WebSocket Server
│   ├── HTTP Server (Control UI, APIs)
│   └── Plugin Registry
│
├── Agents
│   ├── Pi Embedded Runner
│   ├── Model Providers (Anthropic, Google, OpenAI)
│   ├── Auth Profiles & Failover
│   └── Tool Execution
│
├── Sessions
│   ├── Session Index (JSON5)
│   ├── Transcripts (JSONL)
│   └── Delivery Context
│
├── Channels
│   ├── WhatsApp (Baileys)
│   ├── Discord
│   ├── Telegram
│   └── Other plugins
│
├── Cron
│   ├── Heartbeat Polls
│   ├── Wake Events
│   └── Isolated Jobs
│
├── Canvas (Port 18793)
│   └── HTML Display for Nodes
│
└── Nodes
    ├── Mac App
    ├── iOS App
    └── Android App
```

## 🔑 Key Concepts

### Session Keys
Format: `agent:scope:session-id`
- `agent:main:main` - Main direct chat
- `agent:main:discord-chan-123` - Discord channel
- `agent:main:whatsapp-group-456` - WhatsApp group

### Model Failover
1. Primary model (MiniMax-M2 or Claude Sonnet 4.5)
2. Auth error → Next profile
3. Rate limit → Cooldown + next
4. Context overflow → Compact + retry
5. All fail → Fallback model (MiniMax → GLM → Qwen → etc)

### Delivery Context
Routes replies to correct channel/user:
```typescript
{
  channel: "whatsapp",
  to: "+15551234567",
  accountId: "default",
  threadId: null
}
```

## 📂 Source Code Locations

| Component | Directory |
|-----------|-----------|
| **Gateway** | `src/gateway/` |
| **Agents** | `src/agents/` |
| **Sessions** | `src/config/sessions/` |
| **Channels** | `extensions/whatsapp/` (and others) |
| **Cron** | `src/cron/` |
| **Memory** | `src/memory/` |
| **Plugins** | `src/plugins/` |
| **Canvas** | `src/canvas-host/` |

## 🛠️ Technology Stack

**Core:**
- Node.js ≥22
- TypeScript
- pnpm (monorepo)

**Networking:**
- WebSocket (`ws`)
- mDNS/Bonjour
- Tailscale (optional)

**Messaging:**
- WhatsApp: `@whiskeysockets/baileys`
- Discord: `discord.js`
- Telegram: `node-telegram-bot-api`

**AI/ML:**
- Ollama/MiniMax: `@ollama/ollama` (local/cloud models)
- Anthropic: `@anthropic-ai/sdk`
- Google: `@google/generative-ai`
- OpenAI: `openai`

**Storage:**
- Config: JSON5
- Sessions: JSONL
- Memory: Markdown

## 📊 Key Metrics

**Codebase:**
- ~700 lines: Gateway initialization
- ~700 lines: Agent runtime
- ~470 lines: Session storage
- ~500 lines: WhatsApp channel
- ~300 lines: Cron scheduler

**File Counts:**
- `src/gateway/`: ~100 files
- `src/agents/`: ~250 files
- `extensions/`: ~10 channel plugins
- `skills/`: ~50 bundled skills

## 🎯 Quick Navigation

**Want to understand:**
- **Messaging flow?** → Start with [04-channels-whatsapp.md](04-channels-whatsapp.md)
- **AI execution?** → Read [02-agent-runtime.md](02-agent-runtime.md)
- **Session storage?** → Check [03-sessions-management.md](03-sessions-management.md)
- **Scheduled tasks?** → See [05-cron-scheduler.md](05-cron-scheduler.md)
- **Overall system?** → Begin with [00-architecture-overview.md](00-architecture-overview.md)

## 📖 Documentation Features

Each architecture document includes:
- **File locations** - Exact source file paths
- **Line numbers** - Function/class definitions
- **Key functions** - Signatures and purposes
- **Module connections** - Import/export relationships
- **Technology selection** - Why specific libraries chosen
- **Data flows** - Request/response patterns
- **Code examples** - Real TypeScript snippets

## 🔍 Research Methodology

1. **Source exploration** - Read actual TypeScript source
2. **Pattern identification** - Extract key abstractions
3. **Flow tracing** - Follow data through system
4. **Function cataloging** - Document entry points
5. **Integration mapping** - Connect components

## 🚀 Next Deep Dives

**Potential additions:**
- Agent tools architecture
- Skills system internals
- Memory search implementation
- Model catalog structure
- Ollama provider integration
- Molt integration details
- Control UI architecture
- Mobile app structure

## 📝 Notes

- Documentation extracted from OpenClaw v2026.4.2
- Based on source at ` \openclaw`
- Focus on practical architecture patterns
- Includes source code line references
- Updated: 2026-04-05

---

*This documentation was created by systematically exploring the OpenClaw codebase and extracting architectural patterns, key functions, and module relationships.*

---

> **Disclaimer:** This is unofficial community documentation created for educational purposes. It is not affiliated with, endorsed by, or maintained by the OpenClaw project. For official documentation and support, please visit https://github.com/openclaw/openclaw
