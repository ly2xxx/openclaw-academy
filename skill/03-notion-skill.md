# Notion Skill

**Location:** ` \openclaw\skills\notion\`  
**Emoji:** 📝  
**Requirements:** `NOTION_API_KEY` environment variable  
**Homepage:** https://developers.notion.com

## Purpose
Create/read/update pages, databases (data sources), and blocks in Notion using the API.

## Setup

1. Create integration at https://notion.so/my-integrations
2. Copy API key (starts with `ntn_` or `secret_`)
3. Store it:
   ```bash
   mkdir -p ~/.config/notion
   echo "ntn_your_key_here" > ~/.config/notion/api_key
   ```
4. Share pages/databases with integration ("..." → "Connect to")

## API Basics

All requests need:
```bash
NOTION_KEY=$(cat ~/.config/notion/api_key)
curl -X GET "https://api.notion.com/v1/..." \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json"
```

**Note:** `Notion-Version: 2025-09-03` header is REQUIRED

## Key Operations

### Search
```bash
curl -X POST "https://api.notion.com/v1/search" \
  -d '{"query": "page title"}'
```

### Get Page Content
```bash
curl "https://api.notion.com/v1/blocks/{page_id}/children"
```

### Create Page in Database
```bash
curl -X POST "https://api.notion.com/v1/pages" \
  -d '{
    "parent": {"database_id": "xxx"},
    "properties": {
      "Name": {"title": [{"text": {"content": "New Item"}}]},
      "Status": {"select": {"name": "Todo"}}
    }
  }'
```

### Query Database
```bash
curl -X POST "https://api.notion.com/v1/data_sources/{data_source_id}/query" \
  -d '{
    "filter": {"property": "Status", "select": {"equals": "Active"}},
    "sorts": [{"property": "Date", "direction": "descending"}]
  }'
```

## Property Types

- **Title:** `{"title": [{"text": {"content": "..."}}]}`
- **Rich text:** `{"rich_text": [{"text": {"content": "..."}}]}`
- **Select:** `{"select": {"name": "Option"}}`
- **Multi-select:** `{"multi_select": [{"name": "A"}, {"name": "B"}]}`
- **Date:** `{"date": {"start": "2024-01-15"}}`
- **Checkbox:** `{"checkbox": true}`
- **Number:** `{"number": 42}`
- **URL:** `{"url": "https://..."}`
- **Relation:** `{"relation": [{"id": "page_id"}]}`

## 2025-09-03 Version Changes

**Databases → Data Sources:**
- Use `/data_sources/` endpoints for queries
- Two IDs: `database_id` (for parent) and `data_source_id` (for queries)
- Search returns `"object": "data_source"`

## Notes

- IDs are UUIDs (with or without dashes)
- Rate limit: ~3 requests/second
- Cannot set view filters via API (UI-only)

## Source
SKILL.md: ` \openclaw\skills\notion\SKILL.md`
