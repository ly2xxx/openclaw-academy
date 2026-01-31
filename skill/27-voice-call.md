# Voice Call - Phone Call Integration

**Location:** ` \openclaw\skills\voice-call\`  
**Emoji:** 📞  
**Requirements:** voice-call plugin enabled  
**Skill Key:** `voice-call`

## Purpose
Start and manage voice calls via OpenClaw using Twilio, Telnyx, Plivo, or mock provider.

## Setup

Enable in config (`~/.openclaw/openclaw.json`):
```json
{
  "plugins": {
    "entries": {
      "voice-call": {
        "enabled": true,
        "config": {
          "provider": "twilio",
          "fromNumber": "+15555550100",
          "twilio": {
            "accountSid": "AC...",
            "authToken": "..."
          }
        }
      }
    }
  }
}
```

## Providers

### Twilio
```json
{
  "provider": "twilio",
  "fromNumber": "+15555550100",
  "twilio": {
    "accountSid": "AC...",
    "authToken": "..."
  }
}
```

### Telnyx
```json
{
  "provider": "telnyx",
  "fromNumber": "+15555550100",
  "telnyx": {
    "apiKey": "KEY...",
    "connectionId": "..."
  }
}
```

### Plivo
```json
{
  "provider": "plivo",
  "fromNumber": "+15555550100",
  "plivo": {
    "authId": "...",
    "authToken": "..."
  }
}
```

### Mock (Development)
```json
{
  "provider": "mock"
}
```

## CLI Usage

**Initiate call:**
```bash
openclaw voicecall call --to "+15555550123" --message "Hello from OpenClaw"
```

**Check status:**
```bash
openclaw voicecall status --call-id <id>
```

## Tool Actions

### initiate_call
```json
{
  "action": "initiate_call",
  "message": "Hello, this is an automated call",
  "to": "+15555550123",
  "mode": "default"
}
```

### continue_call
```json
{
  "action": "continue_call",
  "callId": "call-123",
  "message": "Here's more information..."
}
```

### speak_to_user
```json
{
  "action": "speak_to_user",
  "callId": "call-123",
  "message": "Thank you for your input"
}
```

### end_call
```json
{
  "action": "end_call",
  "callId": "call-123"
}
```

### get_status
```json
{
  "action": "get_status",
  "callId": "call-123"
}
```

## Best Practices

1. **Verify phone numbers** before calling
2. **Use mock provider** for testing
3. **Track call IDs** for follow-up actions
4. **Handle status checks** to monitor call state
5. **Respect call duration limits** based on provider

## Source
SKILL.md: ` \openclaw\skills\voice-call\SKILL.md`
