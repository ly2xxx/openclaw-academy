# Chrome Extension Setup - Step-by-Step

## What This Does
Allows Helpful Bob to read and interact with Chrome tabs you explicitly attach (for Gmail monitoring, web tasks, etc.).

---

## Installation (One-Time)

### 1. Check if Extension is Already Installed
- Open Chrome
- Look in the toolbar (top-right) for the **Clawdbot icon** (looks like a claw/lobster 🦞)
- If you see it → skip to **Usage** below
- If not → continue to step 2

### 2. Install the Extension
**Option A: From Chrome Web Store (Recommended)**
- Visit: https://chromewebstore.google.com/detail/openclaw-browser-relay/nglingapjinhecnfejdcpihlpneeadjp
- Click **"Add to Chrome"**
- Click **"Add extension"** when prompted

**Option B: Developer Mode (if not in store yet)**
- Open Chrome
- Go to: `chrome://extensions/`
- Enable **"Developer mode"** (toggle in top-right)
- Click **"Load unpacked"**
- Navigate to: `~\.clawdbot\browser-extension\` (or wherever Clawdbot installed it)
- Click **"Select Folder"**

---

## Usage (Every Time)

### How to Attach a Tab

1. **Open the tab you want me to access**
   - Example: https://mail.google.com for committee email

2. **Click the Clawdbot extension icon** in Chrome toolbar
   - Icon: 🦞 (claw/lobster)
   - Location: Top-right of Chrome window

3. **Badge turns ON**
   - You'll see the badge change (icon lights up or shows "ON")
   - Tab is now attached and I can see/control it

4. **That's it!**
   - Tell me "Gmail tab is attached" or just trigger a heartbeat
   - I can now read inbox, open emails, etc.

### How to Detach a Tab

- Click the extension icon again → badge turns OFF
- Or just close the tab
- I lose access immediately when detached

---

## For Gmail Committee Monitoring

### Setup Tonight:
1. Open Chrome
2. Go to https://mail.google.com
3. Make sure you're logged into the **Holm Street Owners Committee** account
4. Click the Clawdbot extension icon → attach the tab
5. Leave Chrome running in the background

### What Happens Next:
- During heartbeats (2-4x per day), I'll check the inbox automatically
- I'll flag urgent emails (building issues, time-sensitive items)
- I'll summarize non-urgent emails daily
- You can detach anytime by clicking the icon again

---

## Privacy & Control

✅ **You control which tabs I see** - only attached tabs are accessible
✅ **No monitoring when detached** - clicking icon OFF = instant privacy
✅ **No background recording** - I only look when you ask or during scheduled checks
✅ **Close Chrome = full disconnect** - closing browser stops all access

---

## Troubleshooting

**"Extension not showing in toolbar"**
- Go to `chrome://extensions/`
- Find "Clawdbot Browser Relay"
- Make sure it's **enabled** (toggle on)
- Click the puzzle piece icon in toolbar → Pin the extension

**"Can't find the extension in Web Store"**
- Use Developer Mode method (step 2, Option B above)
- Check: `~\.clawdbot\browser-extension\`

**"Badge won't turn ON"**
- Refresh the page (F5)
- Try clicking the icon again
- Check if Clawdbot Gateway is running: `clawdbot gateway status`

---

## Questions?

Just ask me via WhatsApp when you're setting it up tonight. I'll guide you through any issues.

Ready when you are! 🦞
