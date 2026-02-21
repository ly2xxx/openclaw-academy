# Clawdbot → OpenClaw Migration Guide

**For:** Upgrading from Clawdbot 2026.1.24-3 to OpenClaw 2026.2.13+

**Date:** 2026-02-14  
**Updated:** 2026-02-21 (Completed - Windows Migration)

---

## ✅ MIGRATION COMPLETED

*This section is a real-world walkthrough of a Windows migration executed by an AI assistant — kept as a reference example with sensitive details redacted.*

**System:** Windows 10.0.22631 (x64)  
**Started:** 2026-02-21 09:58 GMT  
**Executed by:** AI Assistant

### ✅ Completed Steps 

#### Step 1: Pre-Migration Assessment ✅
- **Node version:** v22.14.0 ✅ (exceeds v22.12.0 minimum — OpenClaw is a Node.js application; Node is required regardless of install method, including the PowerShell installer)
- **Clawdbot version:** 2026.1.24-3 (installed via npm global)
- **State directory:** `~user\.clawdbot` ✅
- **Workspace:** `~user\clawd` ✅
- **WhatsApp:** Connected and paired ✅

#### Step 2: Comprehensive Backup ✅ (prompt CLAWDBOT to create backup)
- **Backup location:** `~user\clawdbot-migration-backup-YYYYMMDD-HHMMSS`
- **Backup size:** ~871 MB (11,707 files)
- **State directory:** ✅ Backed up (`.clawdbot/` → `backup/.clawdbot/`)
- **Workspace:** ✅ Backed up (`clawd/` → `backup/clawd-workspace/`)
- **Manifest:** ✅ Created (`BACKUP-MANIFEST.md`)

**Critical files preserved:**
- `clawdbot.json` (config + backup versions)
- WhatsApp credentials (`devices/whatsapp/`)
- API keys (Anthropic, OpenAI, Gemini, Brave Search)
- Session history (`agents/main/sessions/`)
- Memory logs (daily + MEMORY.md)
- Cron jobs

---

### 🎯 Step-by-Step Guide

#### Step 3: Stop Clawdbot Gateway

```powershell
clawdbot gateway stop
```

**Wait for confirmation:** "Gateway stopped"

**If gateway is still running (check with `netstat -ano | findstr "18789"`):**

```powershell
# Find the gateway process
netstat -ano | findstr "18789"
# Note the PID (last column)

# Force stop the process (replace <PID> with actual PID)
Stop-Process -Id <PID> -Force

# Or disable scheduled task first, then stop
schtasks /Change /TN "Clawdbot Gateway" /DISABLE
Stop-Process -Id <PID> -Force
```

**Verify it's stopped:** Try sending yourself a message — if Clawdbot doesn't respond, it's truly stopped.

#### Step 4: Install OpenClaw

**Option A: PowerShell Installer (Recommended for Windows)**

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

**Benefits:**
- Official Windows installer
- Handles environment setup automatically
- Better PATH configuration
- May auto-run doctor migrations

**Option B: npm (Manual Control)**

```powershell
npm install -g openclaw@latest
```

**Use this if:**
- You prefer manual control
- PowerShell script has issues
- You want to skip auto-wizard

**Option C: Bash Script (if WSL/Git Bash available)**

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

> ⚠️ **DO NOT USE BUN** for the gateway runtime — known bugs with WhatsApp/Telegram.

**Verify installation:**

```powershell
openclaw --version
```

Expected: `2026.2.13` or higher

**Note:** If the PowerShell installer auto-runs the wizard or doctor, you can skip or exit it — we'll run doctor manually after copying the state directory.

#### Step 5: Migrate State Directory

**On Windows, create `.openclaw` and copy from `.clawdbot`:**

```powershell
# Create new state directory
New-Item -ItemType Directory -Path "~user\.openclaw" -Force

# Copy everything from .clawdbot to .openclaw
Copy-Item -Path "~user\.clawdbot\*" -Destination "~user\.openclaw\" -Recurse -Force

# Rename the main config file
Rename-Item "~user\.openclaw\clawdbot.json" "openclaw.json"
```

🚧⚠️ Tip: Ignore PowerShell errors — will be covered by Verify steps below.

**Verify critical files copied:**

```powershell
Get-ChildItem ~user\.openclaw -Force | Select-Object Name
```

**Expected to see:**
- `openclaw.json` (renamed from `clawdbot.json`)
- `agents/`
- `credentials/`
- `devices/`
- `cron/`
- `memory/`
- `browser/`
- `identity/`
- `subagents/`

#### Step 6: Run OpenClaw Doctor 🏥

**This is the CRITICAL migration step — always use `--fix` to apply changes:**

```powershell
openclaw doctor --fix
```

> ⚠️ Running `openclaw doctor` **without** `--fix` only diagnoses — it won't apply any repairs. Always use `--fix` after a migration.

**What it does:**
- Migrates deprecated config keys
- Updates channel policies
- Repairs service entrypoints (incl. device scope upgrades)
- Runs security audit
- Checks for common misconfigurations

**Follow all prompts carefully** — doctor will guide you through fixes.

**Expected output:**
- Config migrations applied
- Path fixes (if needed)
- Security checks passed
- Ready to restart

> ℹ️ **Known warning:** *"State dir migration skipped: target already exists"* — this appears if you manually copied `.clawdbot` to `.openclaw` before running doctor. It is expected and safe to ignore **if** you already verified all files are present (Step 5 above). See Troubleshooting → [Scope mismatch / "pairing required" on loopback](#issue-pairing-required-on-loopback-scope-mismatch) if you hit CLI auth issues after this.

#### Step 7: Restart Gateway

```powershell
openclaw gateway restart
```

**OR** (if you want to install as a service):

```powershell
openclaw gateway install
openclaw gateway start
```

#### Step 8: Verification Checklist

**A. Check Gateway Status:**

```powershell
openclaw status
```

**Expected:** Gateway running, state dir = `~user\.openclaw`, workspace = `~user\clawd`

**B. Run Health Checks:**

```powershell
openclaw health
```

**Expected:** All checks pass (or warnings you recognise)

**C. Verify WhatsApp Still Connected:**

```powershell
# Send test message to yourself
openclaw message send --to +1XXXXXXXXXX --message "Migration test - OpenClaw is alive!"
```

**Expected:** Message delivers to WhatsApp without re-pairing

**D. Check Session History:**

```powershell
openclaw sessions list
```

**Expected:** Previous sessions visible

**E. Verify Workspace Files:**

```powershell
Get-ChildItem ~user\clawd | Select-Object Name
```

**Expected:** AGENTS.md, SOUL.md, USER.md, MEMORY.md, memory/, etc.

**F. Test a Chat:**

Open WhatsApp and send yourself a message. Verify your AI assistant responds correctly.

---

### 📝 Migration Notes (AI Handoff)

**What the AI did:**

1. ✅ Assessed current setup (Node v22.14.0, Clawdbot 2026.1.24-3)
2. ✅ Located state directory (`~user\.clawdbot`)
3. ✅ Created comprehensive backup (~871 MB, 11,707 files)
4. ✅ Documented everything in `BACKUP-MANIFEST.md`
5. ✅ Updated this migration guide with Windows-specific steps
6. ⏸️ **LEFT RUNNING** — Clawdbot gateway kept active so WhatsApp stayed connected during handoff

**What YOU need to do:**

1. Read this guide section (you're doing it now ✅)
2. Review the backup manifest: `~user\clawdbot-migration-backup-YYYYMMDD-HHMMSS\BACKUP-MANIFEST.md`
3. When ready, execute **Step 3** (stop Clawdbot)
4. Follow **Steps 4–8** sequentially
5. Run verification checks
6. Once confirmed working, you can clean up old Clawdbot (but keep backup for 7+ days!)

**Safety notes:**

- Your backup is comprehensive and complete ✅
- WhatsApp credentials are preserved ✅
- Original `.clawdbot` directory remains untouched ✅
- You can rollback if needed (see BACKUP-MANIFEST.md)
- API keys are all in the config ✅

**Expected migration time:** 10–15 minutes (if everything goes smoothly)

**If you hit issues:**

1. Check OpenClaw logs: `openclaw logs --follow`
2. Re-run doctor: `openclaw doctor --verbose`
3. Restore from backup if needed (instructions in BACKUP-MANIFEST.md)
4. Ask your AI assistant for help!

---

### 🎬 Migration State Summary

| Item | Status |
|------|--------|
| Backup | ✅ Complete (~871 MB) |
| Clawdbot Gateway | ✅ Stopped |
| OpenClaw Installed | ✅ Done |
| State Migrated | ✅ Done |
| Doctor Run | ✅ Done |
| Verification | ✅ Done |

---

## ⚠️ Before You Start

**Critical facts:**
- OpenClaw IS Clawdbot (rebranded/evolved)
- Requires **Node ≥22.12.0** (check: `node --version`) — OpenClaw is a Node.js app; this applies even if you install via the PowerShell script
- Migration preserves: config, auth, sessions, channel logins, workspace files
- **BACKUP FIRST** — state dir contains secrets (API keys, WhatsApp creds, OAuth tokens)

---

## 🐛 Troubleshooting

### Issue: "Pairing required" on loopback — scope mismatch {#issue-pairing-required-on-loopback-scope-mismatch}

**Symptom:**
```
gateway connect failed: Error: pairing required
Error: gateway closed (1008): pairing required
Gateway target: ws://127.0.0.1:18789
reason: scope-upgrade
```

**Cause:** Device credentials migrated from Clawdbot are missing the `operator.read` scope that OpenClaw's gateway requires. The gateway rejects the CLI even on loopback until scopes are corrected.

This happens when:
- You manually copied `.clawdbot` → `.openclaw` *before* running `openclaw doctor --fix`
- Doctor skipped the state migration because `.openclaw` already existed

**Fix:**

```powershell
# 1. Stop the gateway
openclaw gateway stop

# 2. Patch paired.json — add operator.read to all device scopes
$paired = Get-Content "~user\.openclaw\devices\paired.json" | ConvertFrom-Json
foreach ($deviceId in $paired.PSObject.Properties.Name) {
    $device = $paired.$deviceId
    if ($device.scopes -notcontains "operator.read") {
        $device.scopes = @($device.scopes) + "operator.read"
    }
    foreach ($role in $device.tokens.PSObject.Properties.Name) {
        $token = $device.tokens.$role
        if ($token.scopes -notcontains "operator.read") {
            $token.scopes = @($token.scopes) + "operator.read"
        }
    }
}
$paired | ConvertTo-Json -Depth 10 | Set-Content "~user\.openclaw\devices\paired.json" -Encoding UTF8

# 3. Clear any stale pending repair requests
'{}' | Set-Content "~user\.openclaw\devices\pending.json" -Encoding UTF8

# 4. Delete the stale client-side auth token so CLI re-authenticates fresh
Remove-Item "~user\.openclaw\identity\device-auth.json" -Force -ErrorAction SilentlyContinue

# 5. Restart gateway
openclaw gateway start

# 6. Verify
openclaw health
```

**Expected after fix:** `openclaw health` shows WhatsApp linked, agents listed, session store visible — no more 1008 errors.

---

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

- **Simple migration:** 10–15 minutes
- **With troubleshooting:** 30–60 minutes
- **First-time + learning:** 1–2 hours

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

**Last updated:** 2026-02-21  
**OpenClaw version:** 2026.2.13+  
**Clawdbot version:** 2026.1.24-3  

---

*Good luck! The migration is straightforward if you follow the steps. Remember: backup first, doctor always.* 🦞
