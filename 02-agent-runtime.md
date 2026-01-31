# Agent Runtime Architecture

**Directory:** `src/agents/`  
**Language:** TypeScript  
**Core File:** `pi-embedded-runner/run.ts`

## Purpose

The Agent Runtime executes AI model requests with:
- Auth profile management & rotation
- Model failover on errors
- Context window management
- Tool execution coordination
- Session compaction (memory management)

## Core Function

```typescript
runEmbeddedPiAgent(params: RunEmbeddedPiAgentParams) → EmbeddedPiRunResult
```

**Location:** `src/agents/pi-embedded-runner/run.ts` (line ~74)

## Key Responsibilities

### 1. Model Resolution (lines 88-109)
```typescript
resolveModel(provider, modelId, agentDir, config)
```

- Resolves provider/model pair (e.g., `anthropic/claude-sonnet-4-5`)
- Validates model exists in registry
- Checks context window size
- Returns model config + auth storage

**Models:**
- `src/agents/model-catalog.ts` - Model registry
- `src/agents/model-selection.ts` - Selection logic

### 2. Context Window Guard (lines 111-134)
```typescript
evaluateContextWindowGuard(info, warnBelowTokens, hardMinTokens)
```

- **Warn threshold:** `CONTEXT_WINDOW_WARN_BELOW_TOKENS` = 8000 tokens
- **Hard minimum:** `CONTEXT_WINDOW_HARD_MIN_TOKENS` = 4000 tokens
- Blocks requests if context too small
- Triggers failover if model inadequate

**File:** `src/agents/context-window-guard.ts`

### 3. Auth Profile Management (lines 136-158)

```typescript
resolveAuthProfileOrder(cfg, store, provider, preferredProfile)
```

**Profile rotation strategy:**
1. Check preferred profile (if user-specified)
2. Try profiles in order (round-robin)
3. Skip profiles in cooldown (rate limit protection)
4. Failover to next profile on auth errors

**Key functions:**
- `ensureAuthProfileStore()` - Load auth profiles (line ~136)
- `resolveAuthProfileOrder()` - Determine profile order (line ~147)
- `markAuthProfileFailure()` - Put profile in cooldown
- `markAuthProfileGood()` - Mark profile as working
- `isProfileInCooldown()` - Check cooldown status

**Files:**
- `src/agents/auth-profiles.ts` - Profile management
- `src/agents/model-auth.ts` - Auth resolution

### 4. Failover Logic (lines 159-192)

**Failover triggers:**
- Auth errors → Try next profile
- Rate limits → Cooldown + next profile
- Context overflow → Compact session or failover
- Timeout → Retry or failover
- Unknown errors → Failover if configured

**Failover reasons:**
```typescript
type FailoverReason =
  | "auth"           // Authentication failed
  | "rate_limit"     // Rate limit hit
  | "context"        // Context overflow
  | "timeout"        // Request timeout
  | "unknown"        // Other errors
```

**Key functions:**
- `classifyFailoverReason()` - Determine error type (line ~172)
- `isAuthAssistantError()` - Check if auth error
- `isRateLimitAssistantError()` - Check if rate limit
- `isContextOverflowError()` - Check if context overflow

**File:** `src/agents/failover-error.ts`

### 5. Session Compaction (context overflow handling)

When context window fills up:
1. Try to compact session (remove old messages)
2. If compaction fails → trigger failover
3. Retry with compacted session

**Key function:**
```typescript
compactEmbeddedPiSessionDirect(sessionId, agentDir, config)
```

**File:** `src/agents/pi-embedded-runner/compact.ts`

### 6. Attempt Execution (main loop)

**Retry loop structure:**
```typescript
while (profileIndex < profileCandidates.length) {
  try {
    // Get API key for current profile
    apiKeyInfo = getApiKeyForModel(...)
    
    // Mark profile as used
    markAuthProfileUsed(authStore, profileId)
    
    // Execute attempt
    result = await runEmbeddedAttempt(...)
    
    // Mark profile as good
    markAuthProfileGood(authStore, profileId)
    
    return result
  } catch (error) {
    // Classify error
    if (isAuthError) {
      markAuthProfileFailure(...)
      profileIndex++ // Try next profile
      continue
    }
    if (isContextOverflow) {
      // Try compaction
      await compactEmbeddedPiSessionDirect(...)
      continue // Retry with compacted session
    }
    // ... other error handling
  }
}
```

**Key function:**
```typescript
runEmbeddedAttempt(attemptParams) → AttemptResult
```

**File:** `src/agents/pi-embedded-runner/run/attempt.ts`

## Key Modules

### Tool Execution

**File:** `src/agents/tools/`
- `bash-tools.ts` - Shell command execution
- `channel-tools.ts` - Messaging tools
- `openclaw-tools.ts` - OpenClaw-specific tools

**Tool split:**
```typescript
splitSdkTools(tools) → { sdkTools, customTools }
```

Separates SDK-native tools from custom implementations.

**File:** `src/agents/pi-embedded-runner/tool-split.ts`

### System Prompt

```typescript
createSystemPromptOverride(params) → string
```

Generates system prompt with:
- Agent identity (name, emoji, role)
- Workspace context (files, skills)
- Tool descriptions
- Memory hooks

**File:** `src/agents/pi-embedded-runner/system-prompt.ts`

### Sandbox Info

```typescript
buildEmbeddedSandboxInfo(params) → SandboxInfo
```

Provides sandbox metadata to agent:
- Workspace directory
- Allowed commands
- Docker access
- Environment variables

**File:** `src/agents/pi-embedded-runner/sandbox-info.ts`

## Concurrency Management

### Lanes (Queues)

**Session lane:** One request at a time per session
**Global lane:** Cross-session rate limiting

```typescript
resolveSessionLane(sessionKey) → string
resolveGlobalLane(lane?) → string
```

Prevents concurrent runs in same session.

**File:** `src/agents/pi-embedded-runner/lanes.ts`

### Run Registry

**Tracks active runs:**
```typescript
isEmbeddedPiRunActive(sessionId) → boolean
isEmbeddedPiRunStreaming(sessionId) → boolean
abortEmbeddedPiRun(sessionId) → void
waitForEmbeddedPiRunEnd(sessionId) → Promise<void>
```

**File:** `src/agents/pi-embedded-runner/runs.ts`

## Message Flow

```
User Message
  ↓
Queue in Session Lane
  ↓
Resolve Model + Auth Profile
  ↓
Check Context Window
  ↓
Build Payload (messages + tools)
  ↓
Run Attempt (API call)
  ↓
  ├─ Success → Return result
  └─ Error → Classify
      ├─ Auth Error → Next profile
      ├─ Rate Limit → Cooldown + next profile
      ├─ Context Overflow → Compact + retry
      └─ Other → Failover or error
```

## Technology Stack

- **Language:** TypeScript
- **Model SDKs:**
  - `@anthropic-ai/sdk` - Claude models
  - `@google/generative-ai` - Gemini models
  - OpenAI SDK - GPT models
- **Concurrency:** Command queue with lanes
- **Auth Storage:** JSON file (`~/.openclaw/agents/<id>/auth-profiles.json`)
- **Session Storage:** JSONL files (`~/.openclaw/agents/<id>/sessions/<session-id>.jsonl`)

## Important Files

| File | Purpose |
|------|---------|
| `pi-embedded-runner/run.ts` | Main agent runner |
| `pi-embedded-runner/attempt.ts` | Single API call attempt |
| `pi-embedded-runner/compact.ts` | Session compaction |
| `auth-profiles.ts` | Auth profile management |
| `model-auth.ts` | API key resolution |
| `model-catalog.ts` | Model registry |
| `model-selection.ts` | Model selection logic |
| `failover-error.ts` | Failover error handling |
| `context-window-guard.ts` | Context window checks |
| `usage.ts` | Token/cost tracking |

## Error Handling

**Error classification:**
```typescript
isAuthAssistantError(error) → boolean       // Auth failure
isRateLimitAssistantError(error) → boolean  // Rate limit
isContextOverflowError(error) → boolean     // Context too large
isFailoverErrorMessage(msg) → boolean       // Failover trigger
isTimeoutErrorMessage(msg) → boolean        // Timeout
```

**File:** `src/agents/pi-embedded-helpers.ts`

## Next Steps

- Explore session management (`src/sessions/`)
- Understand model providers (`src/providers/`)
- Review tool execution (`src/agents/tools/`)
