# OpenClaw — Codebase Overview & Gateway Startup

## Technology Stack

OpenClaw is a **Node.js** application written entirely in **TypeScript**. If you are coming from a Python or Java background, here is what each piece of the stack does:

| Technology | Version | Role |
|---|---|---|
| **Node.js** | ≥ 22.14.0 | JavaScript runtime — executes the compiled TypeScript |
| **TypeScript** | ^6.0.2 | Typed superset of JavaScript; compiled to JS before running |
| **pnpm** | 10.32.1 | Package manager (like `pip` or `maven`, but for Node) |
| **Hono** | 4.12.10 | Lightweight HTTP server framework (handles REST routes) |
| **Express** | ^5.2.1 | Secondary HTTP framework (used by some plugin routes) |
| **ws** | ^8.20.0 | WebSocket server library (real-time UI ↔ gateway communication) |
| **Zod** | ^4.3.6 | Runtime schema validation for config files and API inputs |
| **ES Modules** | — | `"type": "module"` in package.json — uses `import/export`, not `require()` |

### TypeScript → JavaScript compilation

The source lives in `src/` as `.ts` files. Before running, TypeScript compiles everything into `dist/` as plain `.js` files. When you see a stack trace pointing to `dist/`, the matching source is the same relative path under `src/`.

### Project layout

```
src/                  Core gateway source (TypeScript)
  cli/                CLI entry points and argument parsing
  gateway/            HTTP server, WebSocket server, health monitor
  agents/             LLM agent runner, model selection, fallback logic
  cron/               Scheduled job store and executor
  config/             Config loading, schema, defaults
  infra/              Low-level utilities (heartbeat, bonjour/mDNS, logging)
  channels/           Messaging channel adapters
extensions/           Optional integrations (whatsapp, telegram, ollama, anthropic, …)
plugins/              Plugin contract types and loader
dist/                 Compiled JS output (generated, do not edit)
```

---

## Gateway Startup Sequence

The sequence below maps each line from the startup log to the exact source file and line responsible for it.

### Phase 1 — Process bootstrap

**File:** [`src/entry.ts:200`](src/entry.ts)

```
openclaw gateway start
```

`entry.ts` is the binary entry point (registered as `"bin": { "openclaw": "openclaw.mjs" }` in `package.json`). The function `runMainOrRootHelp()` at line 200 is the first user-space code to run. At line 204 it dynamically imports and calls `runCli()` from `src/cli/run-main.ts`, which dispatches the `gateway` subcommand to `src/cli/gateway-cli/run.ts`.

---

### Phase 2 — Configuration loading

**File:** [`src/cli/gateway-cli/run.ts:284`](src/cli/gateway-cli/run.ts)

```
[gateway] loading configuration…
```

`loadConfig()` is called to read and parse `openclaw.json` from the user state directory (`C:\Users\vl\.openclaw\openclaw.json` on your machine). The JSON is validated against the Zod schema defined in `src/config/schema.base.generated.ts`. Default values are merged in by `applyModelDefaults()` in `src/config/defaults.ts`.

---

### Phase 3 — Authentication resolution

**File:** [`src/cli/gateway-cli/run.ts:399`](src/cli/gateway-cli/run.ts)

```
[gateway] resolving authentication…
```

The auth mode is determined (none / token / password / trusted-proxy) and credentials are prepared. This is also where the warning `"Gateway is binding to a non-loopback address"` is emitted if the bind host is not `127.0.0.1`.

---

### Phase 4 — Gateway process start

**File:** [`src/cli/gateway-cli/run.ts:507`](src/cli/gateway-cli/run.ts)

```
[gateway] starting...
```

Control passes to `startGatewayServer()` in `src/gateway/server.impl.ts`. From here the gateway is initialising its runtime state — allocating port, setting up middleware, wiring routes.

---

### Phase 5 — HTTP server bind

**File:** [`src/gateway/server.impl.ts:735`](src/gateway/server.impl.ts)

```
[gateway] starting HTTP server...
```

The Hono HTTP server binds to the configured address (default `0.0.0.0:18789`). The WebSocket upgrade handler (using the `ws` library) is attached to the same port — there is no separate WebSocket port. All browser ↔ gateway communication goes through this single port.

---

### Phase 6 — Canvas UI host mount

**File:** [`src/gateway/server-runtime-state.ts:129`](src/gateway/server-runtime-state.ts)

```
[canvas] host mounted at http://0.0.0.0:18789/__openclaw__/canvas/ (root C:\Users\vl\.openclaw\canvas)
```

The control UI static assets are served from the `canvas` path. The UI is the web interface you open in the browser at `http://127.0.0.1:18789`.

---

### Phase 7 — MCP loopback server

**File:** [`src/gateway/server.impl.ts:888`](src/gateway/server.impl.ts)

```
[gateway] MCP loopback server listening on http://127.0.0.1:{port}/mcp
```

A separate **Model Context Protocol** (MCP) server starts on a random loopback port. MCP is an open standard (by Anthropic) that lets the agent expose tools to external clients such as Claude Code. The port changes on every restart, which is why the log shows a dynamic value like `54121`.

---

### Phase 8 — Heartbeat runner

**File:** [`src/infra/heartbeat-runner.ts:1157`](src/infra/heartbeat-runner.ts)

```
[heartbeat] started
```

A periodic timer fires to check that configured agents are still alive and within their scheduled active hours. If an agent has `activeHours` set in config, the heartbeat enforces those windows.

---

### Phase 9 — Channel health monitor

**File:** [`src/gateway/channel-health-monitor.ts:200`](src/gateway/channel-health-monitor.ts)

```
[health-monitor] started (interval: 300s, startup-grace: 60s, channel-connect-grace: 120s)
```

A watchdog that polls every 5 minutes and restarts any messaging channel (e.g. WhatsApp) that has gone stale, disconnected, or stopped. Key thresholds defined in `src/gateway/channel-health-policy.ts`:

| Threshold | Value | Meaning |
|---|---|---|
| Check interval | 300 s | How often the monitor polls |
| Startup grace | 60 s | Gateway is not monitored immediately after start |
| Channel connect grace | 120 s | New channels are given 2 min before being flagged |
| Stale socket | 30 min | No events for 30 min → channel restarted |
| Max restarts | 10 / hour | Prevents restart thrashing |

---

### Phase 10 — Gateway ready

**File:** [`src/gateway/server-startup-log.ts:25–35`](src/gateway/server-startup-log.ts)

```
[gateway] agent model: ollama/minimax-m2.7:cloud
[gateway] ready (6 plugins, 10.8s)
[gateway] log file: \tmp\openclaw\openclaw-2026-05-17.log
```

Three lines are emitted back-to-back from `server-startup-log.ts`:
- **line 25** — the primary LLM model resolved from `agents.defaults.model.primary`
- **line 33** — plugin count and total startup duration since `entry.ts` was invoked
- **line 35** — path to the current log file (rotates daily)

---

### Phase 11 — Channels and sidecars

**File:** [`src/gateway/server.impl.ts:1410`](src/gateway/server.impl.ts)

```
[gateway] starting channels and sidecars...
```

`startGatewaySidecars()` iterates through the configured messaging channels and starts each one as a child process or in-process provider. For WhatsApp (`extensions/whatsapp/`), this spawns the WhatsApp Web connection. Sidecar processes such as the browser controller also start here.

---

### Phase 12 — Hook handlers loaded

**File:** [`src/gateway/server-startup.ts:142`](src/gateway/server-startup.ts)

```
[hooks] loaded 4 internal hook handlers
```

Internal hooks (e.g. `gateway_start`, `message_received`) are registered. These are the event triggers that OpenClaw fires at runtime — plugins and user-defined scripts subscribe to them.

---

### Phase 13 — Plugin runtime backend

```
[plugins] embedded acpx runtime backend registered (cwd: C:\Users\vl\clawd)
[plugins] embedded acpx runtime backend ready
```

The `acpx` runtime (the plugin execution sandbox) registers itself and then signals readiness. The two-step log ("registered" then "ready") reflects an async initialisation — registered means the plugin is known to the gateway, ready means it has completed its own startup handshake.

---

### Phase 14 — WhatsApp channel live

```
[whatsapp] Listening for personal WhatsApp inbound messages.
```

Emitted by the WhatsApp provider in `extensions/whatsapp/` once the WhatsApp Web session is authenticated and the connection is receiving messages.

---

## Key Runtime Files (post-startup)

| Concern | File |
|---|---|
| WebSocket connection handling | `src/gateway/server/ws-connection.ts` |
| Session persistence (JSON store) | `src/config/sessions/store.ts` |
| Session write lock | `src/agents/session-write-lock.ts` |
| LLM idle timeout | `src/agents/pi-embedded-runner/run/llm-idle-timeout.ts` |
| Model fallback chain | `src/agents/model-fallback.ts` |
| Cron job store | `src/cron/store.ts` → `C:\Users\vl\.openclaw\cron\jobs.json` |
| Bonjour / mDNS advertiser | `src/infra/bonjour.ts` |

---

## User State Directory

All runtime state is stored under `C:\Users\vl\.openclaw\`:

```
.openclaw/
  openclaw.json          Main configuration file
  cron/
    jobs.json            Scheduled cron jobs (safe to delete when gateway is stopped)
    jobs.json.bak        Automatic backup written before each save
  agents/
    main/
      sessions/
        sessions.json    Conversation history
        sessions.json.lock  File-based write lock (auto-removed on clean shutdown)
  canvas/                Control UI static assets
```
