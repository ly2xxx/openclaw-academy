# Day 6 — Cron System & UI

**Goal:** Understand scheduled job execution (the other way the agent runs, besides
chat) and how the browser UI communicates with the gateway. These are two independent
subsystems that reinforce the patterns from previous days.

---

## Part A: Cron System

### Concept: Isolated agent turns

When the gateway starts a cron job, it does something similar to a chat turn but
*without a user sitting at a keyboard*. The job has:
- A **schedule** (cron expression like `0 9 * * *` = every day at 9am)
- A **payload** (what to tell the agent)
- An optional **delivery** (which channel to send the reply to)

The agent runs in an "isolated" lane (separate from the main chat session) so the
cron job cannot interfere with an active conversation.

---

### Step 1 — Job data model
**File:** `src/cron/types-shared.ts`

Read the `CronJobBase` generic type. The generics let different job variants
share a common base while varying the payload and schedule types:

```typescript
interface CronJobBase<
  TSchedule,       // cron expression | fixed delay | one-shot
  TSessionTarget,  // isolated | shared session key
  TWakeMode,       // wake on schedule | wake on message
  TPayload,        // what to tell the agent
  TDelivery,       // where to send the reply
  TFailureAlert    // how to notify on failure
> {
  id:             string;
  agentId:        string;
  name:           string;
  enabled:        boolean;
  schedule:       TSchedule;
  payload:        TPayload;
  delivery?:      TDelivery;
  failureAlert?:  TFailureAlert;
}
```

**`TPayload`** for your flight-check cron was:
```json
{ "kind": "agentTurn", "message": "Execute the PowerShell script..." }
```

---

### Step 2 — Job store
**File:** `src/cron/store.ts`

You already know this file from deleting `jobs.json`. Revisit the code:

**`loadCronStore(storePath)`** (line 216; there is also a synchronous
`loadCronStoreSync()` at line 276) — the `ENOENT` handling pattern you saw earlier:
```typescript
} catch (err) {
  if (err?.code === "ENOENT") return { version: 1, jobs: [] };  // missing = empty
  throw err;  // real errors propagate
}
```
**TypeScript concept:** `err?.code` — optional chaining on a caught `unknown` type.
Caught errors in TypeScript are typed as `unknown` (safer than `any`), requiring
you to check the type before accessing properties.

**`saveCronStore(storePath, store)`** — uses atomic write:
1. Write to a temp file (`jobs.json.tmp`)
2. Rename temp file to `jobs.json`

Renaming is *atomic* on Linux/macOS (one syscall). On Windows it's more complex —
look for the Windows-specific retry logic in this file.

---

### Step 3 — Scheduler service
**File:** `src/cron/service.ts` (the `CronService` class, line 13) and
`src/cron/service/state.ts` (the actual scheduling state)

The scheduler is a class, `CronService`, whose live state is built by
`createCronServiceState()` in `service/state.ts`. Conceptually it:
1. Loads all jobs from `store.ts` on startup
2. For each enabled job, sets a timer for the next run
3. When the timer fires, runs the job — delegating to the isolated agent runner

**TypeScript concept:** `class CronService implements CronServiceContract` — the class
implements a typed *contract* interface, so the rest of the codebase depends on the
interface, not the concrete class (the same dependency-injection idea from Day 3 & 5).

**Cron expression parsing:**
`0 9 * * *` = "at minute 0, hour 9, every day, every month, every weekday"
The `src/cron/schedule.ts` file computes the next `Date` when the job should run.

---

### Step 4 — The isolated agent turn
**File:** `src/cron/isolated-agent/run.ts`

`runCronIsolatedAgentTurn(job, config, deps)` is the cron equivalent of
`runEmbeddedPiAgent()` from Day 4. The difference:

| | Chat turn | Cron turn |
|--|-----------|-----------|
| Trigger | User message | Timer |
| Session | Shared (main chat) | Isolated (new each run, or dedicated) |
| Delivery | WS → UI | Channel (e.g. WhatsApp) |
| Model | From config defaults | From job payload (if set) |
| Timeout | `idleTimeoutSeconds` | `AGENT_TURN_SAFETY_TIMEOUT_MS` (60 min) |

**`delivery-dispatch.ts`** — after the agent responds, this routes the output to
the configured channel. This is why your heartbeat job was trying to send to
"Master Yang" — the delivery was set to WhatsApp with that contact name.

---

### Step 5 — Delivery failure handling
**File:** `src/cron/isolated-agent/delivery-dispatch.ts` (the transient patterns array)

```typescript
const TRANSIENT_DIRECT_CRON_DELIVERY_ERROR_PATTERNS = [
  /gateway timeout/i,
  /gateway not connected/i,
  /gateway closed \(1006/i,
  /econnreset|etimedout/i,
  // ...
];
```

If delivery fails with a transient error, the job is retried. If it fails permanently
(e.g. unknown WhatsApp contact), the failure is logged and the job continues on its
next scheduled run.

**This is exactly what happened with "Master Yang":** the contact name couldn't be
resolved to a phone number, so delivery failed permanently every 30 minutes. It was
a permanent error, not a transient one — retrying doesn't help.

---

## Part B: UI (Control Interface)

### Concept: Lit Web Components

The OpenClaw UI (`ui/`) uses **Lit** — a lightweight library for building
*Web Components*. Web Components are a browser-native standard; Lit is just a
thin layer that makes them easier to write.

Unlike React (which has a virtual DOM), Lit updates only the parts of the DOM
that actually changed. It is also closer to the browser platform — no JSX, no
special compiler.

---

### Step 6 — UI architecture
**File:** `ui/src/ui/app.ts`

The root component. Notice the Lit pattern:

```typescript
@customElement('openclaw-app')     // registers as <openclaw-app> HTML element
class OpenClawApp extends LitElement {

  @state() sessions: Session[] = [];   // reactive state — changes trigger re-render

  render() {
    return html`
      <div class="sidebar">
        ${this.sessions.map(s => html`<session-item .session=${s}></session-item>`)}
      </div>
    `;
  }
}
```

**`@customElement`** — decorator that registers the class as a custom HTML element.
**`@state()`** — decorator that makes a property reactive (like `useState` in React).
**`html\`...\``** — tagged template literal that Lit uses to create DOM efficiently.

---

### Step 7 — Gateway client (WS protocol)
**File:** `ui/src/ui/gateway.ts`

`GatewayBrowserClient` manages the WebSocket connection:

```typescript
class GatewayBrowserClient {
  private socket: WebSocket;
  private pending = new Map<string, (result) => void>();  // id → resolver

  async call(method: string, params: unknown): Promise<unknown> {
    const id = generateId();
    this.socket.send(JSON.stringify({ id, method, params }));

    return new Promise((resolve) => {
      this.pending.set(id, resolve);   // store resolver for later
    });
  }

  // When a message arrives:
  onMessage(raw: string) {
    const msg = JSON.parse(raw);
    if (msg.id && this.pending.has(msg.id)) {
      this.pending.get(msg.id)(msg.result);   // resolve the promise
      this.pending.delete(msg.id);
    }
  }
}
```

**`Map<string, (result) => void>`** — pending requests indexed by ID.
This is the *correlation ID* pattern — standard in any async RPC system.

---

### Step 8 — Controllers (state management)
**File:** `ui/src/ui/controllers/agents.ts` (pick one controller to read)

Controllers in Lit are reusable pieces of state logic:
```typescript
class AgentsController implements ReactiveController {
  private host: ReactiveControllerHost;

  constructor(host) {
    this.host = host;
    host.addController(this);    // subscribe to host lifecycle
  }

  hostConnected() {
    // start polling or subscribing to WS events
  }

  hostDisconnected() {
    // clean up
  }
}
```

This is the *composition* alternative to *inheritance* — rather than extending a base
class with all the state logic, each concern is a separate controller that plugs into
any Lit component.

---

## TypeScript Patterns Introduced Today

| Pattern | Example | Meaning |
|---------|---------|---------|
| Generic interface | `CronJobBase<T1, T2, ...>` | Reusable shape with variable parts |
| `unknown` error type | `catch (err: unknown)` | Safer than `any` — must check type before use |
| Tagged template literal | `html\`<div>\`` | Template string processed by a function |
| Decorator | `@customElement`, `@state()` | Metadata annotation (compile-time transform) |
| Correlation ID | `Map<id, resolver>` | Match async responses to requests |

---

## Key Questions for Day 6

1. What is the difference between `isolated` and `shared` session targets for a cron job?
   When would you use each?
2. Why does `saveCronStore()` write to a temp file first, then rename, rather than
   writing directly to `jobs.json`?
3. The UI sends `sessions.list` over WebSocket. How does `GatewayBrowserClient` match
   the response back to the right caller when multiple requests are in-flight?
4. What is the difference between Lit's `@state()` and a plain class property?
5. Your heartbeat cron job sends "HEARTBEAT_OK" to "Master Yang". Which file in
   `src/cron/` controls whether a permanent delivery failure retries or gives up?

---

## Exercise (Cron)
Look at `src/cron/service/timeout-policy.ts`. What is `AGENT_TURN_SAFETY_TIMEOUT_MS`?
Compare it to the `idleTimeoutSeconds` setting. Explain why a cron job needs *two*
different timeouts.

## Exercise (UI)
Open DevTools while the UI is running. In the Console, type:
```javascript
document.querySelector('openclaw-app')
```
Explore the component's properties. Find the `sessions` array and look at its structure.
This is the live data your Lit component is rendering.
