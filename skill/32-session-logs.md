# Session Logs - Conversation History Search

**Location:** ` \openclaw\skills\session-logs\`  
**Emoji:** 📜  
**Requirements:** `jq`, `rg` (ripgrep)

## Purpose
Search and analyze your complete session logs (older/parent conversations) using jq.

## Trigger

Use when user asks about:
- Prior chats
- Parent conversations  
- Historical context not in memory files

## Location

Session logs: `~/.openclaw/agents/<agentId>/sessions/`

**Files:**
- `sessions.json` - Index mapping session keys to IDs
- `<session-id>.jsonl` - Full conversation transcript per session

## Structure

Each `.jsonl` file contains:
- `type`: "session" (metadata) or "message"
- `timestamp`: ISO timestamp
- `message.role`: "user", "assistant", or "toolResult"
- `message.content[]`: Text, thinking, or tool calls
- `message.usage.cost.total`: Cost per response

## Common Queries

### List Sessions by Date

```bash
for f in ~/.openclaw/agents/<agentId>/sessions/*.jsonl; do
  date=$(head -1 "$f" | jq -r '.timestamp' | cut -dT -f1)
  size=$(ls -lh "$f" | awk '{print $5}')
  echo "$date $size $(basename $f)"
done | sort -r
```

### Find Sessions from Date

```bash
for f in ~/.openclaw/agents/<agentId>/sessions/*.jsonl; do
  head -1 "$f" | jq -r '.timestamp' | grep -q "2026-01-06" && echo "$f"
done
```

### Extract User Messages

```bash
jq -r 'select(.message.role == "user") | .message.content[]? | select(.type == "text") | .text' <session>.jsonl
```

### Search Assistant Responses

```bash
jq -r 'select(.message.role == "assistant") | .message.content[]? | select(.type == "text") | .text' <session>.jsonl | rg -i "keyword"
```

### Total Session Cost

```bash
jq -s '[.[] | .message.usage.cost.total // 0] | add' <session>.jsonl
```

### Daily Cost Summary

```bash
for f in ~/.openclaw/agents/<agentId>/sessions/*.jsonl; do
  date=$(head -1 "$f" | jq -r '.timestamp' | cut -dT -f1)
  cost=$(jq -s '[.[] | .message.usage.cost.total // 0] | add' "$f")
  echo "$date $cost"
done | awk '{a[$1]+=$2} END {for(d in a) print d, "$"a[d]}' | sort -r
```

### Message and Token Count

```bash
jq -s '{
  messages: length,
  user: [.[] | select(.message.role == "user")] | length,
  assistant: [.[] | select(.message.role == "assistant")] | length,
  first: .[0].timestamp,
  last: .[-1].timestamp
}' <session>.jsonl
```

### Tool Usage Breakdown

```bash
jq -r '.message.content[]? | select(.type == "toolCall") | .name' <session>.jsonl | sort | uniq -c | sort -rn
```

### Search Across All Sessions

```bash
rg -l "phrase" ~/.openclaw/agents/<agentId>/sessions/*.jsonl
```

## Fast Text-Only Search (Low Noise)

```bash
jq -r 'select(.type=="message") | .message.content[]? | select(.type=="text") | .text' ~/.openclaw/agents/<agentId>/sessions/<id>.jsonl | rg 'keyword'
```

## Tips

1. **Append-only JSONL** - One JSON object per line
2. **Large sessions** can be several MB - use `head`/`tail`
3. **sessions.json index** maps providers (discord, whatsapp) to session IDs
4. **Deleted sessions** have `.deleted.<timestamp>` suffix
5. **Filter type=="text"** for human-readable content only

## Use Cases

**Find when topic was discussed:**
```bash
rg -l "topic" ~/.openclaw/agents/main/sessions/*.jsonl | xargs -I{} head -1 {} | jq -r '.timestamp'
```

**Cost analysis last week:**
```bash
for f in ~/.openclaw/agents/main/sessions/*.jsonl; do
  date=$(head -1 "$f" | jq -r '.timestamp' | cut -dT -f1)
  [[ $date > "2026-01-24" ]] && echo "$date: $(jq -s '[.[] | .message.usage.cost.total // 0] | add' "$f")"
done
```

**Most used tools:**
```bash
jq -r '.message.content[]? | select(.type == "toolCall") | .name' ~/.openclaw/agents/main/sessions/*.jsonl | sort | uniq -c | sort -rn | head -10
```

## Source
SKILL.md: ` \openclaw\skills\session-logs\SKILL.md`
