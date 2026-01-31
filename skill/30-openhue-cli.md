# OpenHue CLI - Philips Hue Control

**Location:** ` \openclaw\skills\openhue\`  
**Emoji:** 💡  
**Requirements:** `openhue` CLI, Philips Hue Bridge  
**Homepage:** https://www.openhue.io/cli

## Purpose
Control Philips Hue lights and scenes via Hue Bridge.

## Installation

```bash
brew install openhue/cli/openhue-cli
```

## Setup

**Discover bridges:**
```bash
openhue discover
```

**Guided setup:**
```bash
openhue setup
```

**Note:** Press Hue Bridge button when prompted during setup.

## Read Operations

### List Lights

```bash
openhue get light --json
```

### List Rooms

```bash
openhue get room --json
```

### List Scenes

```bash
openhue get scene --json
```

## Write Operations

### Turn On/Off

```bash
# Turn on
openhue set light <id-or-name> --on

# Turn off
openhue set light <id-or-name> --off
```

### Brightness

```bash
# Set brightness (0-100)
openhue set light <id> --on --brightness 50
```

### Color

```bash
# RGB color
openhue set light <id> --on --rgb #3399FF

# Hex color
openhue set light <id> --on --rgb FF6600
```

### Activate Scene

```bash
openhue set scene <scene-id>
```

## Room Targeting

When light names are ambiguous:

```bash
openhue set light "Desk" --room "Office" --on --brightness 75
```

## Best Practices

1. **Press bridge button** during initial setup
2. **Use JSON output** for scripting
3. **Target by room** when names overlap
4. **Test with one light** before batch operations
5. **Scene IDs** are more reliable than scene names

## Common Use Cases

**Morning routine:**
```bash
openhue set light "Bedroom" --on --brightness 30
openhue set light "Kitchen" --on --brightness 80
```

**Movie time:**
```bash
openhue set scene "Movie Night"
```

**Focus mode:**
```bash
openhue set light "Office" --on --brightness 100 --rgb FFFFFF
```

**Dim all lights:**
```bash
for light in $(openhue get light --json | jq -r '.[].id'); do
  openhue set light $light --on --brightness 20
done
```

**Turn all lights off:**
```bash
for light in $(openhue get light --json | jq -r '.[].id'); do
  openhue set light $light --off
done
```

## Troubleshooting

**Bridge not found:**
- Ensure bridge is on same network
- Try `openhue discover` again
- Check bridge IP manually

**Authentication failed:**
- Press bridge button during setup
- Delete old config and re-setup

**Light not responding:**
- Check light name/ID is correct
- Verify light is powered on physically
- Try controlling via Hue app first

## Source
SKILL.md: ` \openclaw\skills\openhue\SKILL.md`
