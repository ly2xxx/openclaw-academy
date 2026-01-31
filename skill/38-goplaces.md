# GoPlaces - Google Places API CLI

**Location:** ` \openclaw\skills\goplaces\`  
**Emoji:** 📍  
**Requirements:** `goplaces` CLI, `GOOGLE_PLACES_API_KEY`  
**Homepage:** https://github.com/steipete/goplaces

## Purpose
Query Google Places API (New) for text search, place details, resolve, and reviews.

## Installation

```bash
brew install steipete/tap/goplaces
```

## Setup

**Required:**
```bash
export GOOGLE_PLACES_API_KEY="your-api-key"
```

**Optional:**
```bash
export GOOGLE_PLACES_BASE_URL="https://..."  # For testing/proxying
```

## Common Commands

### Text Search

**Basic search:**
```bash
goplaces search "coffee"
```

**Filtered search:**
```bash
goplaces search "coffee" --open-now --min-rating 4 --limit 5
```

**Location bias:**
```bash
goplaces search "pizza" --lat 40.8 --lng -73.9 --radius-m 3000
```

**Pagination:**
```bash
goplaces search "pizza" --page-token "NEXT_PAGE_TOKEN"
```

### Resolve Address

```bash
goplaces resolve "Soho, London" --limit 5
```

### Place Details

```bash
# Basic details
goplaces details <place_id>

# With reviews
goplaces details <place_id> --reviews
```

### JSON Output

```bash
goplaces search "sushi" --json
```

## Search Filters

### Rating

```bash
goplaces search "restaurant" --min-rating 4.0
```

### Open Now

```bash
goplaces search "cafe" --open-now
```

### Price Level

```bash
# 0 = Free
# 1 = Inexpensive
# 2 = Moderate
# 3 = Expensive  
# 4 = Very Expensive
goplaces search "restaurant" --min-price 0 --max-price 2
```

### Type Filter

```bash
goplaces search "food" --type restaurant
```

**Note:** API accepts only one type value (sends first `--type` only).

## Output Formats

**Human-readable (default):**
```bash
goplaces search "coffee"
```

**JSON (scripting):**
```bash
goplaces search "coffee" --json | jq '.'
```

**No color:**
```bash
goplaces search "coffee" --no-color
# Or use environment variable
NO_COLOR=1 goplaces search "coffee"
```

## Common Use Cases

**Find nearby restaurants:**
```bash
goplaces search "restaurant" --lat 40.7128 --lng -74.0060 --radius-m 1000 --min-rating 4
```

**Find open coffee shops:**
```bash
goplaces search "coffee" --open-now --min-rating 4 --limit 10
```

**Get place details with reviews:**
```bash
goplaces details ChIJN1t_tDeuEmsRUsoyG83frY4 --reviews
```

**Resolve address to place:**
```bash
goplaces resolve "1600 Amphitheatre Parkway, Mountain View, CA"
```

**Price-filtered search:**
```bash
goplaces search "lunch" --min-price 0 --max-price 2 --open-now
```

## Response Fields

**Search/Resolve:**
- Place ID
- Name
- Address
- Location (lat/lng)
- Rating
- Price level
- Opening hours
- Types

**Details:**
- All search fields plus:
- Phone number
- Website
- Reviews (with --reviews)
- Photos
- Detailed hours

## Best Practices

1. **Use location bias** for better local results
2. **Filter by rating** for quality results
3. **Check --open-now** for immediate needs
4. **Use JSON** for scripting/automation
5. **Paginate** for large result sets

## Troubleshooting

**API key missing:**
```bash
export GOOGLE_PLACES_API_KEY="your-key-here"
```

**No results:**
- Broaden search query
- Remove filters
- Increase radius-m
- Try different location

**Rate limits:**
- Google Places API has rate/quota limits
- Check Google Cloud Console for usage

## Source
SKILL.md: ` \openclaw\skills\goplaces\SKILL.md`
