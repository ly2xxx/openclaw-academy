# Day 5 — Plugin & Extension Architecture

**Goal:** Understand how OpenClaw achieves extensibility — how WhatsApp, Ollama, Anthropic,
and dozens of other integrations are structured as self-contained units, and how the
plugin system connects them to the core gateway without tight coupling.

---

## Concept: Inversion of Control

The gateway does **not** import WhatsApp directly:
```typescript
// This is NOT how it works:
import { sendWhatsAppMessage } from './whatsapp';
```

Instead, WhatsApp *registers itself* with the gateway at startup:
```typescript
// Inside extensions/whatsapp/src/channel.ts:
export const whatsappPlugin: ChannelPlugin = {
  id: 'whatsapp',
  outbound: { send: async (msg) => { /* WhatsApp Web logic */ } },
  status:   { check: async () => { /* health check */ } },
  // ...
};
```

The gateway only knows about the `ChannelPlugin` *interface*, not the implementation.
This is *Inversion of Control* (IoC) / *Dependency Injection* (DI) — a core design
pattern for extensible systems.

**Interview talking point:** "The channel system uses interface-based polymorphism.
The gateway core only depends on the `ChannelPlugin` interface. Each integration
implements that interface independently."

---

## Concept: Extensions vs Plugins

| | Extensions | Plugins |
|--|-----------|---------|
| Location | `extensions/` directory | External npm packages |
| Bundled? | Yes, compiled into `dist/` | No, loaded at runtime |
| Purpose | Channels (WhatsApp), LLM providers (Ollama) | Custom capabilities, commands |
| SDK | `openclaw/plugin-sdk/*` | Same |

---

## File Sequence

### Step 1 — The channel contract (the interface)
**File:** `src/channels/plugins/types.plugin.ts`

This defines `ChannelPlugin<ResolvedAccount, Probe, Audit>` — the interface every
channel must implement. Read the full shape:

```typescript
interface ChannelPlugin<...> {
  id:           ChannelId;          // unique string ID e.g. 'whatsapp'
  meta:         ChannelMeta;        // display name, icon URL
  capabilities: ChannelCapabilities; // DMs? threads? reactions? file upload?
  outbound:     ChannelOutboundAdapter;  // MUST implement: send messages
  status:       ChannelStatusAdapter;    // MUST implement: health check
  config:       ChannelConfigAdapter;    // read/write channel config
  lifecycle?:   ChannelLifecycleAdapter; // optional: init/reload/shutdown hooks
  setupWizard?: ChannelSetupWizard;      // optional: first-time setup UI
  doctor?:      ChannelDoctorAdapter;    // optional: diagnostics
}
```

**TypeScript concept:** Generic types `<ResolvedAccount, Probe, Audit>` let the same
interface work for any channel without losing type information. WhatsApp uses its own
`WhatsAppAccount` type; Telegram uses `TelegramAccount` — both satisfy `ChannelPlugin`.

---

### Step 2 — A real channel implementation
**File:** `extensions/whatsapp/src/channel.ts`

This is the concrete implementation of `ChannelPlugin` for WhatsApp. Notice how it
exports a single object that satisfies the interface:

```typescript
export const whatsappChannel: ChannelPlugin<WhatsAppAccount, ...> = {
  id: 'whatsapp',
  meta: { name: 'WhatsApp', icon: '...' },
  capabilities: { directMessages: true, groupMessages: true, ... },
  outbound: { send: sendWhatsAppMessage },
  status:   { check: checkWhatsAppHealth },
  // ...
};
```

**Also open:** `extensions/whatsapp/src/inbound.ts` — this is where the `[whatsapp]
Inbound message` log comes from. It listens to the WhatsApp Web WebSocket and
emits events into the gateway.

**And:** `extensions/whatsapp/src/auto-reply/monitor.ts` — the 30-minute inactivity
watchdog lives here (`messageTimeoutMs = 30 * 60 * 1000`, line ~258; the
`"… - restarting connection"` log fires around line 403). This is what recycles the
WhatsApp connection after 30 minutes of silence.

---

### Step 3 — Plugin discovery & loading
**File:** `src/plugins/bundle-manifest.ts`
**File:** `src/plugins/bundled-plugin-metadata.ts`

At build time, all extensions are compiled into the bundle. At runtime, the gateway
discovers them via a manifest. Think of it as a registry:

```typescript
// Conceptually:
const pluginManifest = {
  'whatsapp':  () => import('./extensions/whatsapp/channel.js'),
  'telegram':  () => import('./extensions/telegram/channel.js'),
  'ollama':    () => import('./extensions/ollama/provider.js'),
  // ...
};
```

Each entry is a *lazy import* — the code is only loaded when the channel is actually
needed. This keeps startup fast.

---

### Step 4 — Plugin runtime
**File:** `src/plugins/runtime/index.ts`

`createPluginRuntime()` returns a `PluginRuntime` object — the central registry that
the gateway uses to reach all plugins:

```typescript
interface PluginRuntime {
  channel:  ChannelRegistry;   // find/use channel plugins
  provider: ProviderRegistry;  // find/use LLM provider plugins
  command:  CommandRegistry;   // handle slash commands
  hook:     HookRegistry;      // fire/subscribe to lifecycle events
}
```

The gateway stores one `PluginRuntime` instance in `GatewayRuntimeState` and passes
it (via `GatewayRequestContext`) to every handler that needs it.

---

### Step 5 — Lifecycle hooks
**Files:** `src/hooks/types.ts` (hook metadata) and
`src/hooks/internal-hook-types.ts` + `src/hooks/internal-hooks.ts` (the event taxonomy)

Hooks let plugins (and internal handlers) react to gateway events without the gateway
knowing about them. Events are grouped into a small set of **categories**
(`src/hooks/internal-hook-types.ts`):

```typescript
export type InternalHookEventType =
  "command" | "session" | "agent" | "gateway" | "message";
```

Each category has concrete event shapes in `src/hooks/internal-hooks.ts`, e.g.:
- `GatewayStartupHookEvent`     (category: `gateway`)
- `AgentBootstrapHookEvent`     (category: `agent`)
- `SessionPatchHookEvent`       (category: `session`)
- `MessageReceivedHookEvent` / `MessageSentHookEvent` (category: `message`)
- `MessageTranscribedHookEvent` / `MessagePreprocessedHookEvent`

A `Hook`'s metadata declares which events it handles (see the `events[]` field in
`types.ts`, e.g. `["command:new", "session:start"]`).

**Why hooks instead of direct calls?** The gateway fires a `message`-category event without
knowing or caring how many handlers are listening. New plugins can react to events
without modifying gateway code. This is the *Observer* / *Event Emitter* pattern.

---

### Step 6 — The Ollama provider extension
**File:** `extensions/ollama/`

Compare this to the WhatsApp extension. WhatsApp is a *channel* (messaging).
Ollama is a *provider* (LLM). The interface it implements is different:

```typescript
interface LlmProviderPlugin {
  id: string;                    // 'ollama'
  baseUrl: string;               // 'http://127.0.0.1:11434/v1'
  models: LlmModelDescriptor[];  // list of available models
  complete(params): AsyncIterable<Token>;  // call the model
}
```

The Ollama extension:
1. Reads `models.providers.ollama.baseUrl` from config
2. At the `/v1/models` endpoint, discovers which models are locally available
3. Registers each as an available `LlmModelDescriptor`
4. When the agent requests `ollama/minimax-m2.7:cloud`, routes the request to
   `http://127.0.0.1:11434/v1/chat/completions`

**This explains the 404s you saw:** Ollama's `/v1/models` returned a list that did
not include `minimax-m2.7:cloud` because it hadn't been pulled yet.

---

### Step 7 — Plugin SDK
**File:** `packages/plugin-package-contract/src/index.ts`

External plugins (npm packages) must satisfy a contract. This file defines it:
- Required `package.json` fields: `openclaw.compat.pluginApi`, `openclaw.build.openclawVersion`
- The plugin must export a default export matching the `OpenClawPluginApi` interface

**File:** `src/plugin-sdk/` directory — browse the sub-path exports

The SDK gives external plugin authors:
- `channel-setup.ts` — helpers for building channel wizards
- `channel-lifecycle.ts` — managed lifecycle (auto-restart, error handling)
- `runtime.ts` — access to gateway services at runtime
- `zod.ts` — re-exported Zod (so plugins don't bundle their own copy)

---

## TypeScript Patterns Introduced Today

| Pattern | Example | Meaning |
|---------|---------|---------|
| Interface polymorphism | `ChannelPlugin<T>` | Multiple types satisfy one interface |
| Generic types `<T>` | `ChannelPlugin<WhatsAppAccount>` | Type-parameterised interface |
| Lazy import | `() => import('./channel.js')` | Load module only when needed |
| Observer pattern | `hook.on('event', handler)` | Loose coupling via events |
| Barrel export | `index.ts` re-exporting many files | Single import point for a module |

---

## Key Questions for Day 5

1. What is the difference between a *channel* plugin and a *provider* plugin?
   Give one example of each from the `extensions/` directory.
2. Why does the gateway use an interface (`ChannelPlugin`) rather than importing
   WhatsApp directly? What would be the downside of the direct import?
3. The WhatsApp extension and the Telegram extension both implement `ChannelPlugin`.
   How does TypeScript know they are compatible without them inheriting from a class?
4. What is the `HookRegistry` doing that `EventEmitter` from Node.js core doesn't do?
   (Hint: look at how hooks declare their `events[]` metadata.)
5. Explain what happens step-by-step when you first add a new `extensions/xxx/`
   directory to the codebase. What files need to change for it to be loaded?

---

## Exercise

Open `extensions/telegram/src/channel.ts` and `extensions/whatsapp/src/channel.ts`
side by side. List three things they have in common (both implement the contract)
and three things that differ (each handles platform-specific details differently).
This comparison will help you articulate the interface pattern in an interview.
