# Weather Skill

**Location:** ` \openclaw\skills\weather\`  
**Emoji:** 🌤️  
**Requirements:** `curl` (no API key needed!)  
**Homepage:** https://wttr.in/:help

## Purpose
Get current weather and forecasts using free services (no API keys required).

## Primary: wttr.in

### Quick One-Liner
```bash
curl -s "wttr.in/London?format=3"
# Output: London: ⛅️ +8°C
```

### Compact Format
```bash
curl -s "wttr.in/London?format=%l:+%c+%t+%h+%w"
# Output: London: ⛅️ +8°C 71% ↙5km/h
```

### Full Forecast
```bash
curl -s "wttr.in/London?T"
```

### Format Codes

- `%c` - condition (emoji)
- `%t` - temperature
- `%h` - humidity
- `%w` - wind
- `%l` - location
- `%m` - moon phase

### Tips

**Location formats:**
- City names: `wttr.in/London`
- Multi-word cities: `wttr.in/New+York` (URL-encode spaces)
- Airport codes: `wttr.in/JFK`

**Options:**
- Units: `?m` (metric), `?u` (USCS)
- Time range: `?1` (today only), `?0` (current only)
- PNG output: `curl -s "wttr.in/Berlin.png" -o /tmp/weather.png`

## Fallback: Open-Meteo

Free JSON API for programmatic use:

```bash
curl -s "https://api.open-meteo.com/v1/forecast?latitude=51.5&longitude=-0.12&current_weather=true"
```

**Process:**
1. Find coordinates for city
2. Query API
3. Returns JSON with temp, windspeed, weathercode

**Docs:** https://open-meteo.com/en/docs

## Best Practices

1. **Primary:** Use wttr.in for quick weather checks
2. **Fallback:** Use Open-Meteo for programmatic/structured data
3. **No API keys needed** for either service
4. **URL-encode** city names with spaces

## Source
SKILL.md: ` \openclaw\skills\weather\SKILL.md`
