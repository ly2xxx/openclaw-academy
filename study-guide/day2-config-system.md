# Day 2 — Configuration & Schema System

**Goal:** Understand how `openclaw.json` is read, validated, typed, and made available
throughout the codebase. This pattern (schema → type → load → use) is universal in
TypeScript backend services.

---

## Concept: Zod — runtime validation

TypeScript types only exist at compile time. When you read a JSON file at runtime,
TypeScript cannot check that the file actually matches your type — you need a runtime
validator. **Zod** is the most popular choice in the Node.js ecosystem.

```typescript
import { z } from 'zod';

const PersonSchema = z.object({
  name: z.string(),
  age:  z.number().min(0),
});

type Person = z.infer<typeof PersonSchema>;  // derive the TS type from the schema

const result = PersonSchema.safeParse(someJson);
if (result.success) {
  result.data.name  // TS knows this is a string
} else {
  result.error      // ZodError with field-level messages
}
```

In OpenClaw, Zod validates the entire `openclaw.json` file on every startup.

---

## File Sequence

### Step 1 — Where config lives on disk
**File:** `src/config/io.ts` (lines 1–50)

First understand the path resolution:
- Default config path: `C:\Users\vl\.openclaw\openclaw.json`
- Overridable via `OPENCLAW_STATE_DIR` env var
- `resolveConfigPath()` (in `src/config/paths.ts`, imported into `io.ts` at line 80)
  computes the final path

**Key function to find:** `loadConfig()` at line 2390 (and `getRuntimeConfig()` at
line 2399). It is **synchronous** — it returns `OpenClawConfig`, not a Promise. Notice
the caching pattern:
```typescript
const cached = getRuntimeConfigSnapshot();
if (cached) return cached;               // return cache hit
const fresh = readAndValidate();         // synchronous read + Zod parse
setRuntimeConfigSnapshot(fresh);         // store in cache
return fresh;
```

The canonical config *type* (`OpenClawConfig`) lives in `src/config/types.openclaw.ts`.
`src/config/types.ts` re-exports the per-domain type files. When you want the top-level
config shape, open `types.openclaw.ts`.

**TypeScript concept:** this is a *module-level singleton* — the cache variable lives
at module scope, not inside a class. Very common in Node.js.

---

### Step 2 — The schema entry point
**File:** `src/config/zod-schema.ts`

This assembles the Zod schema for the whole `openclaw.json` file.
Follow the imports at the top of the file to see how it is composed from sub-schemas:

```
zod-schema.ts
  ├── zod-schema.core.ts            (gateway, auth, logging settings)
  ├── zod-schema.agents.ts          (AgentsSchema, bindings, broadcast)
  ├── zod-schema.agent-defaults.ts  (agents.defaults block)
  ├── zod-schema.agent-runtime.ts   (ToolsSchema)
  ├── zod-schema.channels-config.ts (ChannelsSchema — channel config)
  ├── zod-schema.approvals.ts       (approval policy)
  ├── zod-schema.hooks.ts           (hook mappings)
  ├── zod-schema.proxy.ts           (proxy config)
  ├── zod-schema.session.ts         (session maintenance)
  └── zod-schema.sensitive.ts       (secret-marking helpers)
```
(LLM provider/API-key config lives in `zod-schema.providers-core.ts`.)

This *schema composition* pattern — building a big schema from smaller ones — is
how real-world Zod usage looks.

---

### Step 3 — Your own config in the schema
**File:** `src/config/zod-schema.agent-model.ts` (used by `zod-schema.agent-defaults.ts`
at line 49 as `model: AgentModelSchema.optional()`)

This is the schema that validates the `agents.defaults.model` block — the primary model
and fallbacks you configured. `AgentModelSchema` (line 3) is a **union**:
```typescript
export const AgentModelSchema = z.union([
  z.string(),                            // shorthand: just a model id string
  z.object({
    primary:   z.string().optional(),
    fallbacks: z.array(z.string()).optional(),
  }).strict(),                           // or the full object form
]);
```
The union means `model` can be *either* a bare string *or* an object. Your `openclaw.json`
uses the object form:
```json
"model": {
  "primary": "ollama/minimax-m2.7:cloud",
  "fallbacks": ["ollama/gpt-oss:20b-cloud"]
}
```

**TypeScript concept:** `z.union([...])` mirrors a TypeScript union type (`string | {...}`).
`.strict()` makes Zod *reject* unknown keys instead of silently dropping them — so a typo
like `"fallback"` (missing the `s`) becomes a validation error rather than being ignored.

---

### Step 4 — Types derived from schema
**File:** `src/config/types.ts` and `src/config/types.agent-defaults.ts`

```typescript
// types.agent-defaults.ts (line 32)
export type AgentModelEntryConfig = {
  alias?: string;       // optional alias ("?" means it can be undefined)
  // ...other optional fields
};
```

**TypeScript concept:** the `?` makes a property *optional* — it can be present or absent.
Without `?`, TypeScript requires it to always be there.

Note this is declared with `type X = {...}` (a *type alias*), not `interface X {...}`.
Both describe object shapes; this codebase favours `type` aliases because they also compose
unions and intersections, which interfaces cannot.

Also look at how `z.infer<typeof SomeSchema>` is used to derive TypeScript types directly
from Zod schemas — you define the schema once, get the type for free.

---

### Step 5 — Config loading pipeline
**File:** `src/config/io.ts` — follow `loadConfig()` implementation inside `createConfigIO` (lines 1526–1671)

The pipeline is executed step-by-step:

1. **Read File** (`src/config/io.ts:1541`)
   Reads the raw string content of `openclaw.json` synchronously from disk:
   `const raw = deps.fs.readFileSync(configPath, "utf-8");`

2. **Parse JSON5** (`src/config/io.ts:1542`)
   Parses the raw text using JSON5 (a JSON superset allowing comments like `//` and trailing commas):
   `const parsed = deps.json5.parse(raw);`

3. **Include Resolution & Env Substitution** (`src/config/io.ts:1543-1546`)
   - Merges any external configurations referenced via `$include` directives (handled in `src/config/includes.ts`).
   - Resolves environment variables placeholders (e.g. `"${VAR}"`) to process environment values (handled in `src/config/env-substitution.ts`).

4. **Migration** (`src/config/io.ts:1548`)
   Strips deprecated plugin configuration structures (e.g. `plugins.installs`) and migrates them to the local plugin index. Rewrite definitions live in `src/config/legacy.ts`.

5. **Validate via Zod Schema** (`src/config/io.ts:1608`)
   Checks that the migrated configuration fits the Zod schema specifications (defined in `src/config/zod-schema.ts` / validated in `src/config/validation.ts` using `.safeParse()`).

6. **Apply Defaults** (`src/config/io.ts:1651`)
   Populates default configurations (e.g., standard provider fallbacks defined in `src/config/defaults.ts`) for omitted settings:
   `materializeRuntimeConfig(validated.config, "load", { ... })`

7. **Cache & Observe** (`src/config/io.ts:1656` & `src/config/runtime-snapshot.ts:261`)
   Logs configuration hashes for auditing/debugging, and stores the resolved config object in the in-memory cache `runtimeConfigSnapshot` to prevent subsequent disk reads.

---

### Step 6 — Default aliases
**File:** `src/config/defaults.ts` (around line 26 for `DEFAULT_MODEL_ALIASES`;
applied by a loop near line 336)

```typescript
const DEFAULT_MODEL_ALIASES: Readonly<Record<string, string>> = {
  opus:     "anthropic/claude-opus-4-7",
  sonnet:   "anthropic/claude-sonnet-4-6",
  gpt:      "openai/gpt-5.4",
  gemini:   "google/gemini-3.1-pro-preview",
  // ...
};
```

`Record<string, string>` is TypeScript shorthand for `{ [key: string]: string }` —
a dictionary/map where both keys and values are strings. The `Readonly<...>` wrapper
makes the map immutable at compile time (you can't accidentally reassign an entry).

`applyModelDefaults()` merges these into the config *only if* the user hasn't already
set an alias — it never overwrites your explicit config.

---

### Step 7 — Session store
**File:** `src/config/sessions/store.ts`
**Lock file:** `src/agents/session-write-lock.ts`

Sessions are separate from config — they are the conversation history.
Key functions in `store.ts`:
- `getSessionEntry()` / `listSessionEntries()` — read session metadata
- `saveSessionStore()` — persist the whole store to disk
- `updateSessionStore()` / `updateSessionStoreEntry()` / `patchSessionEntry()` /
  `upsertSessionEntry()` — mutate-then-save helpers

The exclusive write lock lives separately in `src/agents/session-write-lock.ts` —
`acquireSessionWriteLock()` (line 726) is the function from the Day 0 logs.

The lock file (`sessions.json.lock`) is created with `fs.open(..., "wx")` (the `wx` flag
= write + fail-if-exists, i.e. `O_EXCL`) to ensure only one process writes at a time —
this is a classic file-system mutex.

---

## TypeScript Patterns Introduced Today

| Pattern | Example | Meaning |
|---------|---------|---------|
| `z.infer<typeof X>` | `type Config = z.infer<typeof ConfigSchema>` | Derive TS type from Zod schema |
| `Record<K, V>` | `Record<string, string>` | Dictionary type |
| Optional `?` | `alias?: string` | Property may be absent |
| `satisfies` operator | `x satisfies SomeType` | Check type without widening |
| Module singleton | Top-level `let cache` | One instance per Node.js process |

---

## Key Questions for Day 2

1. What would happen if you put an invalid value in `openclaw.json` (e.g. a number
   where a string is expected)? Trace the code path that catches it.
2. What is JSON5 and why is it used instead of plain JSON here?
3. Why does `loadConfig()` cache its result? What would break if it re-read the file
   every time it was called?
4. If you wanted to add a brand-new validated field under `agents.defaults`, which Zod
   schema file would you edit — and what does `.strict()` (Step 3) do if a config has a
   key the schema doesn't define?
5. What is the difference between `z.string()` and `z.string().optional()`?

---

## Exercise

Open `C:\Users\vl\.openclaw\openclaw.json` and deliberately add a syntax error
(e.g. a missing quote). Start OpenClaw and observe the error message. Then open
`src/config/io.ts` and find exactly where that error is caught and logged.
