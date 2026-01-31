# Canvas Skill

**Location:** ` \openclaw\skills\canvas\`  
**Purpose:** Display HTML content on connected OpenClaw nodes (Mac app, iOS, Android)

## Use Cases
- Games, visualizations, dashboards
- Generated HTML content
- Interactive demos

## Architecture

```
Canvas Host (HTTP Server) ──→ Node Bridge (TCP) ──→ Node App (WebView)
    Port 18793                   Port 18790           Mac/iOS/Android
```

1. **Canvas Host** - Serves static HTML/CSS/JS from `canvasHost.root` directory
2. **Node Bridge** - Communicates canvas URLs to nodes
3. **Node Apps** - Render content in WebView

## Tailscale Integration

Server binding based on `gateway.bind`:

| Bind Mode | Binds To | Canvas URL Uses |
|-----------|----------|-----------------|
| `loopback` | 127.0.0.1 | localhost only |
| `lan` | LAN interface | LAN IP |
| `tailnet` | Tailscale | Tailscale hostname |
| `auto` | Best available | Tailscale > LAN > loopback |

**Key:** When bound to Tailscale, nodes receive URLs like:
```
http://<tailscale-hostname>:18793/__moltbot__/canvas/file.html
```

## Configuration

`~/.openclaw/openclaw.json`:
```json
{
  "canvasHost": {
    "enabled": true,
    "port": 18793,
    "root": "/Users/you/clawd/canvas",
    "liveReload": true
  },
  "gateway": {
    "bind": "auto"
  }
}
```

### Live Reload
When `liveReload: true` (default):
- Watches root directory for changes (chokidar)
- Injects WebSocket client into HTML
- Auto-reloads canvases on file save

Great for development!

## Actions

| Action | Description |
|--------|-------------|
| `present` | Show canvas with optional target URL |
| `hide` | Hide the canvas |
| `navigate` | Navigate to new URL |
| `eval` | Execute JavaScript in canvas |
| `snapshot` | Capture screenshot |

## Workflow

### 1. Create HTML Content
```bash
cat > ~/clawd/canvas/my-game.html << 'HTML'
<!DOCTYPE html>
<html>
<head><title>My Game</title></head>
<body><h1>Hello Canvas!</h1></body>
</html>
HTML
```

### 2. Find Canvas Host URL

**Check bind mode:**
```bash
cat ~/.openclaw/openclaw.json | jq '.gateway.bind'
```

**Construct URL:**
- **loopback:** `http://127.0.0.1:18793/__moltbot__/canvas/<file>.html`
- **tailnet/auto:** `http://<hostname>:18793/__moltbot__/canvas/<file>.html`

**Find Tailscale hostname:**
```bash
tailscale status --json | jq -r '.Self.DNSName' | sed 's/\.$//'
```

### 3. Find Connected Nodes
```bash
openclaw nodes list
```

Look for Mac/iOS/Android nodes with canvas capability.

### 4. Present Content
```
canvas action:present node:<node-id> target:<full-url>
```

**Example:**
```
canvas action:present node:mac-63599bc4 target:http://peters-mac.ts.net:18793/__moltbot__/canvas/snake.html
```

### 5. Control Canvas
```
canvas action:navigate node:<node-id> url:<new-url>
canvas action:snapshot node:<node-id>
canvas action:hide node:<node-id>
```

## Debugging

### White Screen / Content Not Loading
**Cause:** URL mismatch between server bind and node expectation

**Debug:**
1. Check bind: `cat ~/.openclaw/openclaw.json | jq '.gateway.bind'`
2. Check port: `lsof -i :18793`
3. Test URL: `curl http://<hostname>:18793/__moltbot__/canvas/<file>.html`

**Solution:** Use full hostname matching bind mode, not localhost

### "node required" error
Always specify `node:<node-id>` parameter

### "node not connected" error
Node is offline. Use `openclaw nodes list` to find online nodes

### Content Not Updating
1. Check `liveReload: true` in config
2. Ensure file is in canvas root directory
3. Check watcher errors in logs

## URL Path Structure

```
http://<host>:18793/__moltbot__/canvas/index.html
→ ~/clawd/canvas/index.html

http://<host>:18793/__moltbot__/canvas/games/snake.html
→ ~/clawd/canvas/games/snake.html
```

Path prefix: `/__moltbot__/canvas/` (CANVAS_HOST_PATH constant)

## Tips
- Keep HTML self-contained (inline CSS/JS)
- Use default index.html as test page (has bridge diagnostics)
- Canvas persists until you `hide` or navigate away
- Live reload makes development fast!
- A2UI JSON push is WIP - use HTML files for now

## Source
SKILL.md: ` \openclaw\skills\canvas\SKILL.md`
