# Cron Scheduler Architecture

**Directory:** `src/cron/`  
**Language:** TypeScript  
**Entry:** `src/gateway/server-cron.ts`

## Purpose

The Cron Service manages scheduled tasks:
- Heartbeat polls (periodic checks)
- Wake events (reminders, notifications)
- Scheduled agent jobs (one-shot or recurring)
- Isolated agent runs (separate from main session)

## Core Service

**Class:** `CronService`  
**File:** `src/cron/service.ts`

```typescript
class CronService {
  constructor(opts: {
    storePath: string;
    cronEnabled: boolean;
    enqueueSystemEvent: (text: string, opts?) => void;
    requestHeartbeatNow: () => void;
    runHeartbeatOnce: (opts?) => Promise<void>;
    runIsolatedAgentJob: (params) => Promise<void>;
    log: Logger;
    onEvent: (event) => void;
  })
}
```

## Job Types

### 1. Heartbeat Jobs

**Default heartbeat prompt:**
```
Read HEARTBEAT.md if it exists (workspace context). 
Follow it strictly. Do not infer or repeat old tasks from prior chats. 
If nothing needs attention, reply HEARTBEAT_OK.
```

**Heartbeat runner:**
```typescript
runHeartbeatOnce(opts?: { cfg?, reason?, deps? }) → Promise<void>
```

**File:** `src/infra/heartbeat-runner.ts`

**Heartbeat flow:**
1. Load config
2. Resolve main session key
3. Read HEARTBEAT.md (if exists)
4. Run agent with heartbeat prompt
5. Check response:
   - `HEARTBEAT_OK` → Silent (no notification)
   - Other text → Deliver to user

### 2. Wake Events

**Trigger immediate wake:**
```typescript
requestHeartbeatNow() → void
```

**Wake event API:**
```typescript
enqueueSystemEvent(text: string, opts?: {
  sessionKey?: string;
  agentId?: string;
})
```

**File:** `src/infra/heartbeat-wake.ts`

**Use case:**
```bash
openclaw gateway wake --text "Reminder: Meeting in 10 minutes" --mode now
```

### 3. Isolated Agent Jobs

**Purpose:** Run agent in separate session (not main chat)

```typescript
runCronIsolatedAgentTurn({
  cfg,
  deps,
  job,
  message,
  agentId,
  sessionKey,  // e.g., "cron:job-id"
  lane
}) → Promise<void>
```

**File:** `src/cron/isolated-agent.ts`

**Isolated session benefits:**
- Separate transcript (doesn't pollute main chat)
- Custom context (different agent config)
- One-shot execution (no ongoing conversation)
- Deliver results to main session

## Job Storage

**Store path:** `~/.openclaw/cron/jobs.json`

**Format:**
```json5
{
  "jobs": {
    "job-id": {
      "id": "job-id",
      "enabled": true,
      "text": "Check for new emails",
      "schedule": "*/30 * * * *",  // Every 30 minutes
      "mode": "heartbeat",          // or "isolated"
      "agentId": "main",
      "sessionKey": "agent:main:main",
      "createdAt": 1706713200000,
      "nextRunAtMs": 1706715000000
    }
  }
}
```

**Job fields:**
- `id` - Unique job ID
- `enabled` - Enable/disable flag
- `text` - Job text (for isolated jobs) or wake text
- `schedule` - Cron expression (5-field format)
- `mode` - `"heartbeat"` or `"isolated"`
- `agentId` - Target agent
- `sessionKey` - Target session (for wake events)
- `nextRunAtMs` - Next scheduled run timestamp

**Cron expressions:**
```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6) (Sunday to Saturday)
│ │ │ │ │
* * * * *
```

**Examples:**
- `0 9 * * *` - 9:00 AM daily
- `*/30 * * * *` - Every 30 minutes
- `0 22 * * *` - 10:00 PM daily

**File:** `src/cron/schedule.ts`

## Run Logs

**Log path:** `~/.openclaw/cron/logs/<job-id>.jsonl`

**Format:** JSONL (one JSON object per line)

```json
{
  "ts": 1706713200000,
  "jobId": "job-id",
  "action": "finished",
  "status": "success",
  "summary": "Completed heartbeat check",
  "runAtMs": 1706713200000,
  "durationMs": 1234,
  "nextRunAtMs": 1706715000000
}
```

**Functions:**
```typescript
resolveCronRunLogPath({ storePath, jobId }) → string
appendCronRunLog(logPath, entry) → Promise<void>
```

**File:** `src/cron/run-log.ts`

## Gateway Integration

**Build cron service:**
```typescript
buildGatewayCronService({
  cfg,
  deps,
  broadcast
}) → GatewayCronState
```

**Location:** `src/gateway/server-cron.ts` (line ~18)

**Returns:**
```typescript
type GatewayCronState = {
  cron: CronService;
  storePath: string;
  cronEnabled: boolean;
}
```

## Event Broadcasting

**Cron events:**
```typescript
broadcast("cron", event, { dropIfSlow: true })
```

**Event types:**
- `started` - Job execution started
- `finished` - Job execution completed
- `failed` - Job execution failed
- `scheduled` - Job scheduled for next run
- `disabled` - Job disabled

## Configuration

**Config path:** `~/.openclaw/openclaw.json`

```json5
{
  "cron": {
    "enabled": true,
    "store": "~/.openclaw/cron"
  }
}
```

**Environment override:**
```bash
OPENCLAW_SKIP_CRON=1  # Disable cron entirely
```

## CLI Commands

```bash
# List jobs
openclaw cron list

# Add job
openclaw cron add --text "..." --schedule "0 9 * * *"

# Update job
openclaw cron update <job-id> --text "..." --schedule "*/30 * * * *"

# Remove job
openclaw cron remove <job-id>

# Run job immediately
openclaw cron run <job-id>

# View run history
openclaw cron runs <job-id>

# Wake event (immediate)
openclaw gateway wake --text "Reminder text" --mode now

# Wake event (next heartbeat)
openclaw gateway wake --text "Reminder text" --mode next-heartbeat
```

## Heartbeat State Tracking

**File:** `memory/heartbeat-state.json` (in workspace)

**Example:**
```json
{
  "lastModelNotified": null,
  "lastFailoverAlert": null,
  "lastChecks": {
    "email": 1706713200000,
    "calendar": 1706713100000
  }
}
```

Agents use this to track periodic checks and avoid duplicate notifications.

## Job Execution Flow

```
Cron Timer Fires
  ↓
Load Job from Store
  ↓
Check if Enabled
  ↓
Determine Mode (heartbeat vs isolated)
  ↓
  ├─ Heartbeat Mode:
  │   ├─ Trigger heartbeat runner
  │   └─ Agent runs with HEARTBEAT.md context
  │
  └─ Isolated Mode:
      ├─ Create isolated session (cron:job-id)
      ├─ Run agent with job text
      ├─ Capture result
      └─ Optionally deliver to main session
  ↓
Log Result
  ↓
Schedule Next Run
  ↓
Broadcast Event
```

## Context Message Support

**Add prior messages as context:**
```bash
openclaw cron add \
  --text "Check calendar" \
  --schedule "0 9 * * *" \
  --context-messages 5
```

Includes last 5 messages from main session as context for the job.

## Technology Stack

- **Scheduler:** Custom timer-based (setInterval)
- **Cron parsing:** Standard 5-field cron expressions
- **Storage:** JSON files (jobs.json, logs/*.jsonl)
- **Execution:** Isolated agent runs + heartbeat integration
- **Broadcasting:** WebSocket events to connected clients

## Important Files

| File | Purpose |
|------|---------|
| `cron/service.ts` | CronService class |
| `cron/isolated-agent.ts` | Isolated job execution |
| `cron/schedule.ts` | Cron expression parsing |
| `cron/run-log.ts` | Run history logging |
| `cron/store.ts` | Job storage |
| `gateway/server-cron.ts` | Gateway integration |
| `infra/heartbeat-runner.ts` | Heartbeat execution |
| `infra/heartbeat-wake.ts` | Wake event triggering |

## Best Practices

1. **Use heartbeat mode** for periodic checks (email, calendar)
2. **Use isolated mode** for standalone tasks (reports, summaries)
3. **Set `enabled: false`** for one-shot reminders after firing
4. **Include context** in reminder text (what triggered it)
5. **Track state** in `heartbeat-state.json` to avoid duplicates

## Next Steps

- Explore memory system (`src/memory/`)
- Review model catalog (`src/agents/model-catalog.ts`)
- Study Molt kanban integration (`packages/moltbot/`)
