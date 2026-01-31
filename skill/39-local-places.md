# Local Places - Google Places Proxy

**Location:** ` \openclaw\skills\local-places\`  
**Emoji:** 📍  
**Requirements:** `uv`, `GOOGLE_PLACES_API_KEY`  
**Homepage:** https://github.com/Hyaxia/local_places

## Purpose
Search for nearby places using a local Google Places API proxy server.

## Setup

```bash
cd {baseDir}
echo "GOOGLE_PLACES_API_KEY=your-key" > .env
uv venv && uv pip install -e ".[dev]"
uv run --env-file .env uvicorn local_places.main:app --host 127.0.0.1 --port 8000
```

## Two-Step Flow

1. **Resolve location first** (get coordinates)
2. **Search places** (with location bias)

## Quick Start

### 1. Check Server

```bash
curl http://127.0.0.1:8000/ping
```

### 2. Resolve Location

```bash
curl -X POST http://127.0.0.1:8000/locations/resolve \
  -H "Content-Type: application/json" \
  -d '{"location_text": "Soho, London", "limit": 5}'
```

### 3. Search Places

```bash
curl -X POST http://127.0.0.1:8000/places/search \
  -H "Content-Type: application/json" \
  -d '{
    "query": "coffee shop",
    "location_bias": {"lat": 51.5137, "lng": -0.1366, "radius_m": 1000},
    "filters": {"open_now": true, "min_rating": 4.0},
    "limit": 10
  }'
```

### 4. Get Details

```bash
curl http://127.0.0.1:8000/places/{place_id}
```

## Conversation Flow

1. **Vague location** ("near me") → resolve first
2. **Multiple results** → show numbered list, ask user to pick
3. **Ask preferences:** type, open_now, rating, price_level
4. **Search** with `location_bias` from chosen location
5. **Present results** with name, rating, address, open status
6. **Offer** to fetch details or refine search

## Filter Constraints

| Filter | Constraint |
|--------|------------|
| `filters.types` | **Exactly ONE type** (e.g., "restaurant", "cafe") |
| `filters.price_levels` | Integers 0-4 (0=free, 4=very expensive) |
| `filters.min_rating` | 0-5 in 0.5 increments |
| `filters.open_now` | Boolean |
| `limit` | 1-20 for search, 1-10 for resolve |
| `location_bias.radius_m` | Must be > 0 |

## Response Format

```json
{
  "results": [
    {
      "place_id": "ChIJ...",
      "name": "Coffee Shop",
      "address": "123 Main St",
      "location": {"lat": 51.5, "lng": -0.1},
      "rating": 4.6,
      "price_level": 2,
      "types": ["cafe", "food"],
      "open_now": true
    }
  ],
  "next_page_token": "..."
}
```

**Pagination:** Use `next_page_token` as `page_token` in next request.

## Best Practices

1. **Resolve location first** for vague queries
2. **One type only** in filters
3. **Use location_bias** for better results
4. **Check open_now** for immediate needs
5. **Paginate** for more results

## Source
SKILL.md: ` \openclaw\skills\local-places\SKILL.md`
