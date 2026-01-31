# Trello API

**Location:** ` \openclaw\skills\trello\`  
**Emoji:** 📋  
**Requirements:** `jq` CLI, `TRELLO_API_KEY`, `TRELLO_TOKEN`  
**Homepage:** https://developer.atlassian.com/cloud/trello/rest/

## Purpose
Manage Trello boards, lists, and cards via REST API.

## Setup

1. **Get API key:** https://trello.com/app-key
2. **Generate token:** Click "Token" link on that page
3. **Set environment variables:**
   ```bash
   export TRELLO_API_KEY="your-api-key"
   export TRELLO_TOKEN="your-token"
   ```

## Common Operations

### List Boards

```bash
curl -s "https://api.trello.com/1/members/me/boards?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" | jq '.[] | {name, id}'
```

### List Lists in Board

```bash
curl -s "https://api.trello.com/1/boards/{boardId}/lists?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" | jq '.[] | {name, id}'
```

### List Cards in List

```bash
curl -s "https://api.trello.com/1/lists/{listId}/cards?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" | jq '.[] | {name, id, desc}'
```

### Create Card

```bash
curl -s -X POST "https://api.trello.com/1/cards?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -d "idList={listId}" \
  -d "name=Card Title" \
  -d "desc=Card description"
```

### Move Card to Another List

```bash
curl -s -X PUT "https://api.trello.com/1/cards/{cardId}?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -d "idList={newListId}"
```

### Add Comment to Card

```bash
curl -s -X POST "https://api.trello.com/1/cards/{cardId}/actions/comments?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -d "text=Your comment here"
```

### Archive Card

```bash
curl -s -X PUT "https://api.trello.com/1/cards/{cardId}?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -d "closed=true"
```

## Examples

**Find specific board:**
```bash
curl -s "https://api.trello.com/1/members/me/boards?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" | jq '.[] | select(.name | contains("Work"))'
```

**Get all cards on board:**
```bash
curl -s "https://api.trello.com/1/boards/{boardId}/cards?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" | jq '.[] | {name, list: .idList}'
```

## Rate Limits

- **300 requests per 10 seconds** per API key
- **100 requests per 10 seconds** per token
- `/1/members` endpoints: **100 requests per 900 seconds**

## Notes

1. **Board/List/Card IDs** from Trello URL or list commands
2. **Full account access** - keep API key & token secret!
3. **Use jq** for JSON parsing and filtering

## Source
SKILL.md: ` \openclaw\skills\trello\SKILL.md`
