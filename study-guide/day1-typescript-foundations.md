# Day 1 — TypeScript Foundations & Project Setup

**Goal:** Understand how a large TypeScript monorepo is structured, compiled, and executed.
By the end you should be able to explain how `openclaw gateway start` goes from source
to running process.

---

## Concept: TypeScript in a nutshell

TypeScript adds a *type layer* on top of JavaScript. The types only exist at edit/compile
time — they are completely erased before the code runs. Node.js executes plain `.js` files.

Key ideas to grasp today:
- `interface` / `type` — shapes of data, no runtime cost
- `async/await` — sugar over Promises; Node.js is single-threaded but non-blocking
- `import/export` with `.js` extensions — ES Module system (even though the source is `.ts`)
- Strict mode — `strictNullChecks`, `noImplicitAny` — forces you to handle every edge case

---

## File Sequence

### Step 1 — The binary wrapper
**File:** `openclaw.mjs` (root)

This is the file Node.js actually executes when you run `openclaw`. It is a tiny shim
that sets up the V8 compile cache (speeds up cold starts) and then imports `dist/entry.js`.
Notice it uses `import()` (dynamic import) not `require()` — this is ES Modules.

**What to notice:**
- `.mjs` extension = force ES Module, even without `"type":"module"` in package.json
- `enableCompileCache()` — caches the V8 bytecode so the next start is faster

---

### Step 2 — Project identity
**File:** `package.json` (root, lines 1–40)

```
"type": "module"        → all .js files in this package use import/export
"bin": { "openclaw" }   → this is what gets symlinked into PATH when installed globally
"engines": { "node": ">=22.19.0" }  → minimum Node.js version required
"packageManager": "pnpm@11.2.2"     → lock the package manager version
```

**What to notice:**
- The `exports` field (lines 40–100) — controls what other packages can import from `openclaw`
- Each `"./plugin-sdk/xxx"` entry exposes a sub-path import (like `import x from 'openclaw/plugin-sdk/core'`)

---

### Step 3 — Monorepo layout
**File:** `pnpm-workspace.yaml`

OpenClaw is a *monorepo* — many packages in one repo, managed together.
```yaml
packages:
  - .          # root (the main openclaw package)
  - ui/        # the control web UI
  - packages/* # shared libraries (moltbot, plugin-package-contract, etc.)
  - extensions/* # channel integrations (whatsapp, telegram, ollama, etc.)
```

Each `extensions/xxx/` folder has its own `package.json` and is a separate npm package,
but they all share the same `node_modules` via pnpm workspaces (symlinks).

**Why this matters for interviews:** monorepo vs polyrepo is a common architecture question.
The trade-off: monorepo = atomic commits across packages, shared tooling, but slower CI.

---

### Step 4 — TypeScript compiler config
**File:** `tsconfig.json` (root)

Key settings to understand:

```jsonc
"target": "ES2023"          // compile to modern JS — no need for old polyfills
"module": "NodeNext"        // use Node.js native ES Module resolution
"moduleResolution": "NodeNext" // .js extensions required in imports
"strict": true              // enables all strict checks (null safety, implicit any, etc.)
"noEmit": true              // TS compiler only TYPE-CHECKS — does NOT produce output files
"allowImportingTsExtensions": true // allows `import './foo.ts'` in source
```

**Important:** `noEmit: true` means `tsc` is only used for type checking.
The *actual* compilation to `dist/` is done by the build tool (tsdown).

Path aliases (bottom of tsconfig):
```jsonc
"paths": {
  "openclaw/plugin-sdk/*": ["./src/plugin-sdk/*.js"]
}
```
These let source files write `import x from 'openclaw/plugin-sdk/core'`
instead of a fragile relative `../../../plugin-sdk/core.js`.

---

### Step 5 — The build system
**File:** `tsdown.config.ts`

`tsdown` is an esbuild-based bundler. Unlike `tsc`, it:
- Reads `.ts` source files
- Bundles them (follows imports, inlines code)
- Writes compiled `.js` to `dist/`
- Is very fast (esbuild is written in Go)

Key function: `buildUnifiedDistEntries()` (around line 306; wired into the build config
at line 337)
This produces the entry points for the bundle:
```
src/entry.ts       → dist/entry.js   (the main CLI binary)
src/index.ts       → dist/index.js   (the library API)
src/cli/daemon-cli.ts → dist/...     (the daemon process)
```

**Interview talking point:** "We used tsdown (esbuild-based) for the build because tsc is
too slow for a large monorepo. tsc is still run as a type-check step in CI."

---

### Step 6 — Entry point
**File:** `src/entry.ts` (lines 1–30, then ~282–303)

```typescript
async function runMainOrRootHelp(argv: string[]): Promise<void> {   // line 282
  if (await tryHandleRootHelpFastPath(argv)) return;
  if (await tryHandlePrecomputedCommandHelpFastPath(argv)) return;
  try {
    const { runCli } = await gatewayEntryStartupTrace.measure(       // startup timing trace
      "run-main-import",
      () => import("./cli/run-main.js"),                             // line 292 — dynamic import
    );
    await runCli(argv);
  } catch (error) {
    // formatCliFailureLines(...) prints a friendly failure message
  }
}
```

The function is `async`, and the dynamic import is wrapped in a
`gatewayEntryStartupTrace.measure(...)` call so cold-start timing can be profiled.

**TypeScript concepts here:**
- `string[]` — array of strings type annotation
- `Promise<void>` — a promise that resolves to nothing
- `const { runCli } = await import(...)` — destructure the named export out of the loaded module
- `async/await` with `try/catch` — the modern alternative to `.then().catch()`

---

### Step 7 — CLI routing
**File:** `src/cli/run-main.ts`

This file routes subcommands (`gateway`, `start`, `stop`, etc.) to their handlers.
Think of it as a router — `switch` on `argv[0]` and call the right function.

**What to notice:** how commands are registered as an array of objects, each with:
- `name` — the CLI command string
- `run` — async function that handles it
- `help` — usage text

---

## TypeScript Patterns Introduced Today

| Pattern | Example in codebase | Meaning |
|---------|--------------------|---------| 
| `as const` | `ENTRY_WRAPPER_PAIRS = [...] as const` | Make array read-only and infer literal types |
| Optional chaining | `err?.code === "ENOENT"` | Only access `.code` if `err` is not null/undefined |
| Nullish coalescing | `raw ?? ""` | Use right side if left is null/undefined |
| Type assertion | `parsed as Record<string, unknown>` | Tell TS "trust me, this is this type" |
| Dynamic import | `import("./cli/run-main.js")` | Load a module lazily at runtime |

---

## Key Questions for Day 1

1. What is the difference between `tsc` and `tsdown` in this project?
2. Why does `tsconfig.json` set `noEmit: true`?
3. What does `pnpm-workspace.yaml` allow you to do that a single `package.json` cannot?
4. When you run `openclaw gateway start`, trace the exact call chain from `openclaw.mjs`
   to `src/cli/gateway-cli/run.ts`. Name every file it passes through.
5. Why does `src/entry.ts` use dynamic `import()` instead of a top-level `import`?

---

## Exercise

Open `dist/entry.js` after running `pnpm build`. Compare it to `src/entry.ts`.
Observe: all the TypeScript type annotations are gone. The JavaScript logic is identical.
This is what Node.js actually runs.
