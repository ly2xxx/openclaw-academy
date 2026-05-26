# Gateway Architecture Overview

**File:** `src/gateway/server.impl.ts`  
**Language:** TypeScript  
**Entry Point:** `startGatewayServer()` (line ~137)

## Core Purpose

The Gateway is the central control plane for OpenClaw. It:
- Manages WebSocket connections to clients (agents, nodes, UI)
- Routes messages between channels (WhatsApp, Discord, etc.)
- Coordinates agent sessions and conversations
- Handles tool execution and plugin management
- Manages cron jobs and scheduled tasks
- Provides HTTP endpoints (Control UI, OpenAI-compatible API)

## Initialization Flow

```typescript
startGatewayServer(port = 18789, opts) → GatewayServer
```

### 1. Config Loading & Migration (lines 137-183)
- Read config from `~/.openclaw/openclaw.json`
- Migrate legacy config if needed
- Validate config schema
- Auto-enable plugins based on environment

**Key Functions:**
- `readConfigFileSnapshot()` - Read config file
- `migrateLegacyConfig()` - Upgrade old config format
- `applyPluginAutoEnable()` - Auto-enable plugins

### 2. Core Component Initialization (lines 184-243)
- Initialize diagnostic heartbeat
- Set up subagent registry
- Load plugins (channel plugins, extension plugins)
- Resolve runtime config (bind address, ports, TLS)

**Key Functions:**
- `loadGatewayPlugins()` - Load all plugins (line ~217)
- `resolveGatewayRuntimeConfig()` - Determine bind address, ports (line ~224)
- `createWizardSessionTracker()` - Onboarding wizard state (line ~245)

### 3. Runtime State Creation (lines 247-274)
- Create HTTP server (with optional TLS)
- Create WebSocket server
- Initialize Canvas host (HTML display)
- Set up client connection tracking
- Create chat run state (active conversations)

**Key Functions:**
- `createGatewayRuntimeState()` - Main runtime setup (line ~251)
- Returns: `httpServer`, `wss`, `clients`, `broadcast`, `chatRunState`, etc.

### 4. Node Registry & Discovery (lines 275-310)
- Initialize node registry (paired devices: Mac, iOS, Android)
- Start mDNS/Bonjour discovery
- Set up Tailscale exposure (if enabled)
- Configure remote skills caching

**Key Classes:**
- `NodeRegistry` - Tracks paired nodes (line ~275)
- `startGatewayDiscovery()` - mDNS/Bonjour discovery (line ~291)

### 5. Channel Management (lines 311-318)
- Create channel manager for messaging platforms
- WhatsApp, Discord, Telegram, Signal, etc.
- Each channel runs in own runtime environment

**Key Functions:**
- `createChannelManager()` - Channel lifecycle (line ~311)
- Returns: `startChannels`, `startChannel`, `stopChannel`

### 6. Cron Service (lines ~323)
- Build cron scheduler for scheduled tasks
- Heartbeat polls, reminders, recurring jobs

**Key Functions:**
- `buildGatewayCronService()` - Cron scheduler (line ~323)

### 7. Maintenance Timers (lines ~342)
- Health check interval
- Presence update (connected clients)
- Chat run cleanup (abort orphaned runs)
- Deduplication cleanup

**Key Functions:**
- `startGatewayMaintenanceTimers()` - Background tasks (line ~342)

## Server Structure

```
GatewayServer
  ├─ HTTP Server (port 18789 default)
  │   ├─ WebSocket (ws:// or wss://)
  │   ├─ Control UI (/ui)
  │   ├─ OpenAI Chat Completions (/v1/chat/completions)
  │   └─ OpenResponses API (/v1/responses)
  │
  ├─ Canvas Host Server (port 18793)
  │   └─ HTML/CSS/JS serving for nodes
  │
  ├─ Node Registry
  │   └─ Paired devices (Mac, iOS, Android)
  │
  ├─ Channel Manager
  │   ├─ WhatsApp (Baileys)
  │   ├─ Discord
  │   ├─ Telegram
  │   └─ Other channels
  │
  ├─ Cron Service
  │   └─ Scheduled jobs
  │
  └─ Plugin Registry
      ├─ Channel plugins
      └─ Extension plugins
```

## Key Exports

```typescript
export type GatewayServer = {
  close: (opts?: {
    reason?: string;
    restartExpectedMs?: number | null;
  }) => Promise<void>;
};

export type GatewayServerOptions = {
  bind?: "loopback" | "lan" | "tailnet" | "auto";
  host?: string;
  controlUiEnabled?: boolean;
  openAiChatCompletionsEnabled?: boolean;
  openResponsesEnabled?: boolean;
  auth?: GatewayAuthConfig;
  tailscale?: GatewayTailscaleConfig;
  // ... test options
};
```

## Technology Stack

- **Runtime:** Node.js (≥22)
- **Language:** TypeScript
- **WebSocket:** `ws` library
- **HTTP:** Node.js `http` / `https`
- **Discovery:** mDNS/Bonjour (`bonjour-service`)
- **Config:** JSON5 format
- **Logging:** Structured logging with subsystems

## Important Files

| File | Purpose |
|------|---------|
| `server.impl.ts` | Main server implementation |
| `server-runtime-state.ts` | Runtime state creation |
| `server-runtime-config.ts` | Config resolution |
| `server-channels.ts` | Channel manager |
| `server-cron.ts` | Cron service |
| `server-discovery.ts` | mDNS/Bonjour discovery |
| `server-maintenance.ts` | Background maintenance |
| `node-registry.ts` | Node tracking |

## Module Dependencies

```typescript
// Core dependencies
import { resolveAgentWorkspaceDir } from "../agents/agent-scope.js"
import { loadConfig } from "../config/config.js"
import { createSubsystemLogger } from "../logging/subsystem.js"
import { runOnboardingWizard } from "../wizard/onboarding.js"

// Server components
import { createChannelManager } from "./server-channels.js"
import { buildGatewayCronService } from "./server-cron.js"
import { startGatewayDiscovery } from "./server-discovery-runtime.js"
import { createGatewayRuntimeState } from "./server-runtime-state.js"
```

## Next Steps

Explore individual components:
- Agent runtime (`src/agents/`)
- Session management (`src/sessions/`)
- Channel implementations (`src/channels/`)
- Model providers (`src/providers/`)
- Cron scheduler (`src/cron/`)
- Memory system (`src/memory/`)
