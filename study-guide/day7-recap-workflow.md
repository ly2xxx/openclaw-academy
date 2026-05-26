# Day 7 — Review & Interview Preparation

**Goal:** Consolidate the week's learning into fluent, confident talking points.
Practice explaining architecture decisions, trade-offs, and TypeScript patterns
the way a senior engineer would in an interview.

---

## Architecture Summary (memorise this map)

```
openclaw gateway start
        │
        ▼
  src/entry.ts                          ← binary entry, dynamic import
        │
        ▼
  src/cli/gateway-cli/run.ts            ← load config, resolve auth, start server
        │
        ├── src/config/io.ts            ← JSON5 → Zod validation → typed config
        │
        ▼
  src/gateway/server.impl.ts            ← orchestrates HTTP + WebSocket startup
        ├── src/gateway/server-http.ts          ← native node:http server (no framework)
        ├── src/gateway/mcp-http.ts             ← lazy MCP loopback server
        │
        ├── src/gateway/server/ws-connection.ts   ← JSON-RPC over WebSocket
        ├── src/gateway/server-methods/           ← RPC handlers (sessions, models…)
        ├── src/gateway/channel-health-monitor.ts ← 5-min watchdog
        ├── src/plugins/runtime/index.ts          ← plugin registry
        │       │
        │       ├── extensions/whatsapp/          ← ChannelPlugin impl
        │       ├── extensions/ollama/            ← LlmProvider impl
        │       └── extensions/telegram/ …
        │
        ├── src/agents/pi-embedded.ts             ← agent public API
        │       │
        │       ├── model-fallback.ts             ← primary → fallback chain
        │       ├── session-write-lock.ts         ← file mutex for sessions
        │       └── pi-embedded-runner/run.ts     ← turn orchestration
        │
        └── src/cron/service.ts                  ← scheduled job runner
                │
                └── isolated-agent/run.ts        ← cron agent turn
```

---

## Top 10 Interview Questions — with model answers

### 1. "Walk me through the architecture of OpenClaw."

> "OpenClaw is a multi-channel AI gateway — a Node.js process that sits between
> messaging channels like WhatsApp and LLM providers like Ollama or Claude.
>
> The core is a gateway server built directly on Node's native `http`/`https` modules
> (no web framework) plus the `ws` WebSocket library on the same port. It loads
> configuration via Zod-validated JSON5, then starts a plugin runtime that registers
> channel adapters and LLM providers. Inbound messages arrive through channel adapters
> (WhatsApp Web, Telegram, etc.), get routed to an embedded agent runner, which calls
> the LLM and delivers the response back via the same channel.
>
> Separately, a cron scheduler can trigger agent turns on a timer, useful for monitoring
> tasks and automated workflows. The whole system is TypeScript on Node.js 22, using
> ES modules throughout."

---

### 2. "How does the WebSocket protocol work in your system?"

> "The gateway runs a single HTTP server on port 18789. When the browser opens a
> WebSocket connection, the `ws` library intercepts the HTTP Upgrade header and hands
> off to a dedicated connection handler.
>
> We use a simple JSON-RPC-style protocol: the client sends `{ id, method, params }`,
> and the server responds with `{ id, result }` or pushes events as `{ type: 'event', name, payload }`.
> The correlation ID pattern lets the client match responses to requests even when
> multiple calls are in-flight simultaneously.
>
> Each connection goes through a 10-second auth handshake before being promoted to
> an authenticated client. Unauthenticated connections that time out are closed with
> a 'handshake-timeout' log."

---

### 3. "Explain your model fallback system."

> "The agent has a primary model and an ordered list of fallback candidates, all
> configured in `openclaw.json`. When the primary fails, a fallback chain iterates
> through the candidates.
>
> Each failure is classified before deciding whether to advance: a timeout or 404
> (model not found) triggers a skip to the next candidate; an authentication error
> or bad request is treated as fatal and propagates immediately — retrying the same
> credentials won't help.
>
> The LLM idle timeout uses `Promise.race()` — if no token arrives within N seconds,
> an idle timeout error is thrown, which the fallback chain catches and uses to advance.
> Lowering this timeout prevents a slow model from holding the session write lock for
> extended periods, which was causing our gateway to become unresponsive."

---

### 4. "How do you handle concurrency in a Node.js single-threaded environment?"

> "Node.js is single-threaded but non-blocking. Concurrency is achieved through the
> event loop — I/O operations (network calls, file reads) are handed to the OS and
> the event loop processes other work while waiting.
>
> In OpenClaw, we use lanes to manage multiple simultaneous conversations. Each lane
> is an async execution slot named by session key. The gateway tracks which lanes are
> active and enforces `maxConcurrent` limits.
>
> For exclusive resource access (writing session JSON), we use a file-based mutex —
> a lock file created with the O_EXCL flag, which the OS guarantees is atomic. This
> works even across multiple processes.
>
> `setInterval` for the health monitor and `Promise.race()` for timeouts are the two
> key async patterns — neither blocks the event loop."

---

### 5. "How is the plugin/extension system designed?"

> "The channel system uses interface-based polymorphism. The gateway core defines a
> `ChannelPlugin` interface with adapters for outbound messaging, health checks,
> lifecycle management, and so on. Each integration (WhatsApp, Telegram, Slack…)
> implements this interface independently.
>
> Extensions are compiled into the bundle at build time (they live in `extensions/`).
> External plugins load at runtime from npm packages. Both use the same plugin SDK.
>
> A plugin runtime registry holds all registered plugins and is passed via a request
> context object to every handler — this is dependency injection without a framework.
>
> Lifecycle events (gateway_start, message_received, etc.) use an Observer pattern —
> plugins subscribe to events without the gateway knowing which plugins are listening."

---

### 6. "How does TypeScript help you here versus plain JavaScript?"

> "Three main ways:
>
> First, interface contracts. `ChannelPlugin` is a TypeScript interface — the compiler
> verifies at build time that every extension implements every required method with the
> right signatures. A missing method is a compile error, not a runtime crash.
>
> Second, Zod + TypeScript gives us end-to-end type safety for config. We define a Zod
> schema for `openclaw.json`, validate at runtime, and use `z.infer` to derive the
> TypeScript type. The config object throughout the codebase is fully typed.
>
> Third, discriminated unions for the message protocol. `{ type: 'event' } | { type: 'response' }`
> lets TypeScript narrow the type based on the `type` field — no casting needed."

---

### 7. "You had a production issue where the gateway was freezing. How did you diagnose it?"

> "The logs showed two clues: `[session-write-lock] releasing lock held for 124751ms
> (max=15000ms)` and `FailoverError: LLM request timed out` after ~21 minutes of runtime.
>
> I traced the session write lock to `src/agents/session-write-lock.ts` and found that
> the lock is acquired before the session is saved — which happens after the LLM turn
> completes. So the lock was held for the entire duration of the LLM call.
>
> The root cause was `gpt-oss:120b-cloud` being the fallback model and timing out only
> after the idle timeout (the default at the time) — but the agent-turn safety timeout
> was 60 minutes, so the whole turn took ~21 minutes before failing over.
>
> The fix was to reduce `idleTimeoutSeconds` to 30 in config (`src/agents/pi-embedded-runner/run/llm-idle-timeout.ts`
> reads this value). With a 30-second idle timeout, the lock is released within 30s of
> the model going silent, rather than 21 minutes."

---

### 8. "How does cron scheduling work?"

> "Cron jobs are stored as JSON in `~/.openclaw/cron/jobs.json`. On startup, `src/cron/service.ts`
> loads all enabled jobs, parses their cron expressions (e.g. `0 9 * * *` = daily at 9am),
> and calls `getNextRunTime()` to compute the first trigger. A `setTimeout` fires at that time.
>
> When a job fires, it runs in an isolated agent lane — separate from the main chat session
> so it cannot interfere with active conversations. The agent gets the job payload as its
> user message, runs a full turn, then optionally delivers the response to a configured
> channel (e.g. send result to a WhatsApp number).
>
> Delivery failures are classified as transient (retry) or permanent (log and skip). The
> storage uses atomic writes — temp file + rename — to prevent corruption if the process
> crashes mid-write."

---

### 9. "How are HTTP and WebSocket served together, and why no web framework?"

> "Both run on the same TCP port (18789) off a single native `node:http` server —
> there's no Express or Hono in the gateway. The plain `(req, res)` handler in
> `server-http.ts` serves standard HTTP: static UI assets, REST-style endpoints, and
> auth checks done explicitly at the top of the handler. The `ws` library attaches to
> that same server and handles WebSocket connections by listening for the HTTP
> `Upgrade` event.
>
> The WebSocket layer is used for the real-time control UI — it needs bidirectional
> streaming (agent token-by-token output, push events). REST wouldn't work here because
> HTTP responses are single, complete replies.
>
> Dropping the framework keeps the dependency surface small and the control flow
> explicit — useful for a long-running daemon where you want to reason precisely about
> every request path and avoid middleware-ordering surprises."

---

### 10. "Tell me about a configuration mistake you debugged."

> "I configured two Ollama cloud models as primary and fallback (`minimax-m2.7:cloud`
> and `gpt-oss:20b-cloud`), but both returned 404 errors. I initially assumed the model
> names were wrong, but the issue was that Ollama cloud models still need to be registered
> with the local Ollama instance via `ollama pull`, even though they have no local storage.
>
> I confirmed this by running `ollama list` — neither model appeared. After pulling them,
> the 404s stopped immediately.
>
> I also found a name typo in the config — `'name': 'minimax-2.7:cloud'` was missing the
> `'m'` — and `kimi-k2-thinking` had `maxTokens` set equal to `contextWindow` (262144),
> which would have caused the model to attempt 256K tokens of output per response and
> always time out. Reduced that to 16384."

---

## Patterns Cheat Sheet for Interviews

| Pattern | Where in OpenClaw | Generic name |
|---------|-------------------|--------------|
| Load config once, cache forever | `src/config/io.ts` | Module singleton |
| Build big schema from small schemas | `src/config/zod-schema.ts` | Schema composition |
| Every handler gets a context bag | `GatewayRequestContext` | Dependency injection |
| Plugins register, gateway discovers | `src/plugins/runtime/` | Inversion of control |
| Interface, not concrete class | `ChannelPlugin` | Interface polymorphism |
| Primary + fallback list | `src/agents/model-fallback.ts` | Chain of responsibility |
| Race timer against async operation | `Promise.race()` in llm-idle-timeout | Timeout pattern |
| Write temp, then rename | `src/cron/store.ts` | Atomic file write |
| Exclusive file creation | `session-write-lock.ts` | File-based mutex |
| Short name → full model ID | `src/agents/model-selection.ts` | Alias/registry |
| Periodic background check | `setInterval` in health-monitor | Polling watchdog |
| Callback fired on event | `hook.on('message_received', ...)` | Observer / event emitter |
| Match response to request by ID | `Map<id, resolver>` in `ui/gateway.ts` | Correlation ID |

---

## Final Checklist

Before your interview, make sure you can:

- [ ] Explain the monorepo structure and why it is organised that way
- [ ] Trace a WhatsApp inbound message from the log line to `runEmbeddedPiAgent()`
- [ ] Explain why the session write lock was held for 21 minutes (and the fix)
- [ ] Describe the model fallback chain and what triggers each type of skip
- [ ] Explain the `ChannelPlugin` interface and why it enables extensibility
- [ ] Describe the WebSocket JSON-RPC protocol and the correlation ID pattern
- [ ] Explain what Zod is for and how `z.infer` works
- [ ] Explain `Promise.race()` and where it is used here
- [ ] Describe the cron system: storage → schedule → isolated agent turn → delivery
- [ ] Draw the startup sequence from memory (the 14 phases from `openclaw.md`)
