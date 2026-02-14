# Clawdbot → OpenClaw Migration Guide

**For:** Upgrading from Clawdbot 2026.1.24-3 to OpenClaw 2026.2.13+

**Date:** 2026-02-14

---

## ⚠️ Before You Start

**Critical facts:**
- OpenClaw IS Clawdbot (rebranded/evolved)
- Requires **Node ≥22.12.0** (check: `node --version`)
- Migration preserves: config, auth, sessions, channel logins, workspace files
- **BACKUP FIRST** — state dir contains secrets (API keys, WhatsApp creds, OAuth tokens)

---

## 📋 Pre-Migration Checklist

### 1. Identify Your Current Setup

Run on your **current Clawdbot** installation:

```bash
clawdbot status
```

**Note down:**
- State directory (default: `~/.openclaw/` or `~/.clawdbot/`)
- Profile name (if using `--profile`)
- Workspace location (default: `~/.openclaw/workspace/`)
- Gateway mode (local vs remote)
- Node version

### 2. Check Node Version

```bash
node --version
```

**Required:** v22.12.0 or higher

If lower, upgrade Node first:
- **Windows (WSL2):** `nvm install 22` or download from nodejs.org
- **macOS:** `brew install node@22`
- **Linux:** Use nvm or your package manager

---

## 🛡️ Step 1: Backup Everything

**Stop the gateway first:**

```bash
clawdbot gateway stop
# or
openclaw gateway stop
```

**Create backups:**

```bash
cd ~

# Backup state directory (contains config, auth, sessions, channel state)
tar -czf clawdbot-state-backup-$(date +%Y%m%d).tgz .openclaw .clawdbot

# Backup workspace separately (if outside state dir)
tar -czf clawdbot-workspace-backup-$(date +%Y%m%d).tgz .openclaw/workspace

# Optional: Backup specific config
cp ~/.openclaw/openclaw.json ~/openclaw.json.backup
```

**Store backups securely** (they contain secrets!)

---

## 🔄 Step 2: Install OpenClaw

### Option A: Recommended (Website Installer)

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**Notes:**
- Detects existing install
- Upgrades in place
- Runs `openclaw doctor` automatically
- Add `--no-onboard` to skip wizard

### Option B: npm/pnpm (Manual)

```bash
# Using npm
npm install -g openclaw@latest

# OR using pnpm (recommended)
pnpm add -g openclaw@latest
```

**DO NOT USE BUN** for the gateway runtime (WhatsApp/Telegram bugs).

---

## 🔧 Step 3: Migrate State & Config

### If You Used `~/.clawdbot` (Old Default)

```bash
# Copy everything from .clawdbot to .openclaw
cp -r ~/.clawdbot/* ~/.openclaw/

# Verify critical files copied
ls -la ~/.openclaw/openclaw.json
ls -la ~/.openclaw/credentials/
```

### If You Already Use `~/.openclaw`

**Skip this step** — your state is already in the right place.

### If You Used a Profile (e.g., `--profile work`)

```bash
# Ensure the profile directory exists
# Example: if you used --profile work
cp -r ~/.openclaw-work/* ~/.openclaw/
# OR keep the profile: cp -r ~/.clawdbot-work/* ~/.openclaw-work/
```

**Important:** Use the same profile when running OpenClaw.

---

## 🏥 Step 4: Run Doctor (Migrations)

This is the **most critical step** — it applies all config migrations:

```bash
openclaw doctor
```

**What it does:**
- Migrates deprecated config keys
- Updates channel policies
- Repairs service entrypoints (launchd/systemd)
- Runs security audit
- Checks for common misconfigurations

**Follow all prompts** — doctor will guide you through fixes.

---

## 🚀 Step 5: Restart Gateway

```bash
openclaw gateway restart
```

**OR** if you prefer to install as a service:

```bash
openclaw gateway install
openclaw gateway start
```

---

## ✅ Step 6: Verify Migration

### A. Check Gateway Status

```bash
openclaw status
```

**Expected:** Gateway running, correct state dir, workspace visible.

### B. Run Health Checks

```bash
openclaw health
```

**Expected:** All checks pass (or warnings you recognize).

### C. Test Messaging

```bash
openclaw message send --to +447877585536 --message "Migration test"
```

**Expected:** Message delivers to WhatsApp (or your connected channel).

### D. Check Channels Still Connected

- **WhatsApp:** Should NOT require re-pairing if state copied correctly
- **Telegram/Discord:** Check bot still responds
- **Slack/Signal:** Verify connections

### E. Verify Workspace Files

```bash
ls -la ~/.openclaw/workspace/
cat ~/.openclaw/workspace/MEMORY.md
```

**Expected:** Your files are intact.

### F. Check Session History

```bash
openclaw sessions list
```

**Expected:** Previous sessions visible.

---

## 🐛 Troubleshooting

### Issue: Gateway won't start

**Check logs:**
```bash
openclaw logs --follow
```

**Common fixes:**
- Permissions: `chown -R $USER ~/.openclaw`
- Port conflict: Change port in config or kill conflicting process

### Issue: "Config not found" or "Session file path must be within sessions directory"

**Fix:** Profile/state-dir mismatch

```bash
# Check what doctor sees
openclaw doctor --verbose

# Ensure you're using the right profile
openclaw --profile YOUR_PROFILE gateway status
```

### Issue: Channels disconnected (WhatsApp/Telegram)

**WhatsApp:**
- Check `~/.openclaw/state/whatsapp/creds.json` exists
- If missing, you'll need to re-pair

**Telegram:**
- Check `~/.openclaw/openclaw.json` for `channels.telegram.botToken`

### Issue: "Node version too old"

```bash
nvm install 22
nvm use 22
```

Then retry migration.

### Issue: Missing sessions/history

**Verify state dir copied:**
```bash
ls -la ~/.openclaw/agents/main/sessions/
```

If empty, restore from backup:
```bash
tar -xzf clawdbot-state-backup-YYYYMMDD.tgz
```

---

## 🔐 Security Notes

**Your backups contain secrets:**
- WhatsApp session credentials
- API keys (Anthropic, OpenAI, etc.)
- OAuth tokens
- Channel bot tokens

**Protect them:**
- Encrypt backup files
- Store securely (password manager, encrypted drive)
- Delete old backups after successful migration
- Rotate keys if you suspect exposure

---

## 📚 Post-Migration Checklist

After successful migration:

- [ ] Gateway status shows "running"
- [ ] Channels connected (no re-pair needed)
- [ ] Sessions visible in dashboard
- [ ] Workspace files intact (MEMORY.md, USER.md, etc.)
- [ ] Test message sent successfully
- [ ] Health checks pass
- [ ] Backup files stored securely
- [ ] Old Clawdbot uninstalled (if desired)

---

## 🧹 Optional: Clean Up Old Installation

**Only after confirming everything works!**

```bash
# Remove old npm package (if installed)
npm uninstall -g clawdbot

# Archive old state (don't delete yet!)
mv ~/.clawdbot ~/clawdbot-archived-$(date +%Y%m%d)

# Remove old service (macOS)
launchctl unload ~/Library/LaunchAgents/com.clawdbot.gateway.plist
rm ~/Library/LaunchAgents/com.clawdbot.gateway.plist

# Remove old service (Linux)
systemctl --user stop clawdbot-gateway
systemctl --user disable clawdbot-gateway
```

**Wait at least a week** before permanently deleting archived state.

---

## 📖 Additional Resources

- [OpenClaw Docs](https://docs.openclaw.ai)
- [Updating Guide](https://docs.openclaw.ai/install/updating)
- [Migration Guide](https://docs.openclaw.ai/install/migrating)
- [Doctor Command](https://docs.openclaw.ai/gateway/doctor)
- [FAQ](https://docs.openclaw.ai/help/faq)
- [Discord Support](https://discord.gg/clawd)

---

## 🆘 If You Get Stuck

1. **Re-run doctor:** `openclaw doctor --verbose`
2. **Check logs:** `openclaw logs --follow`
3. **Restore from backup** if needed
4. **Ask in Discord:** [https://discord.gg/clawd](https://discord.gg/clawd)
5. **Review changelog:** Check breaking changes in CHANGELOG.md

---

## ⏱️ Estimated Time

- **Simple migration:** 10-15 minutes
- **With troubleshooting:** 30-60 minutes
- **First-time + learning:** 1-2 hours

---

## 🎯 Success Criteria

Migration is successful when:

✅ `openclaw status` shows gateway running  
✅ WhatsApp/channels respond without re-pairing  
✅ Session history intact  
✅ Workspace files accessible  
✅ Health checks pass  
✅ Test message delivers  

---

**Last updated:** 2026-02-14  
**OpenClaw version:** 2026.2.13+  
**Clawdbot version:** 2026.1.24-3  

---

*Good luck! The migration is straightforward if you follow the steps. Remember: backup first, doctor always.* 🦞
