# APNs Authentication Key Setup — Vern Push Notifications

> [!IMPORTANT]
> This is a one-time setup. The same .p8 key works for development AND production, and across all apps on your Apple Developer account.

## Step 1: Generate the Key

1. Go to [Apple Developer → Keys](https://developer.apple.com/account/resources/authkeys/list)
2. Click the **+** button to create a new key
3. Name it: **Vern Push Notifications**
4. Check ✅ **Apple Push Notifications service (APNs)**
5. Click **Continue** → **Register**
6. **Download the `.p8` file** — Apple only lets you download it ONCE
7. Note the **Key ID** shown on the confirmation page (10-character string like `ABC123XYZ0`)

> [!CAUTION]
> You can only download the .p8 file ONE TIME. If you lose it, you have to revoke and create a new key. Save it somewhere safe.

## Step 2: Gather Your IDs

You need three values:

| Value | Where to find it | Example |
|-------|-------------------|---------|
| **Key ID** | Shown when you created the key (also visible in Keys list) | `ABC123XYZ0` |
| **Team ID** | [Apple Developer → Membership](https://developer.apple.com/account#MembershipDetailsCard) — "Team ID" field | `885845M2A4` |
| **Bundle ID** | Your app's bundle identifier in Xcode | `com.vern.companion` |

## Step 3: Upload to Vern Mac

Copy the `.p8` file to the Vern server:

```bash
# From your Mac (replace the filename with your actual key file)
scp ~/Downloads/AuthKey_ABC123XYZ0.p8 vern:/Users/vern/ai-projects/family-dashboard/server/certs/

# Create the certs directory if it doesn't exist
ssh vern "mkdir -p /Users/vern/ai-projects/family-dashboard/server/certs"
```

## Step 4: Configure Environment Variables

SSH into Vern and add to the `.env` file:

```bash
ssh vern
nano /Users/vern/ai-projects/family-dashboard/server/.env
```

Add these lines (replace with your actual values):

```env
# APNs Push Notification Configuration
APNS_KEY_ID=ABC123XYZ0
APNS_TEAM_ID=885845M2A4
APNS_BUNDLE_ID=com.vern.companion
APNS_KEY_PATH=/Users/vern/ai-projects/family-dashboard/server/certs/AuthKey_ABC123XYZ0.p8
APNS_ENVIRONMENT=development
```

> [!NOTE]
> Use `APNS_ENVIRONMENT=development` while testing via Xcode/TestFlight.
> Switch to `production` only after App Store release.

## Step 5: Restart the Server

```bash
ssh vern
source ~/.nvm/nvm.sh
cd /Users/vern/ai-projects/family-dashboard/server
npx pm2 restart vern-server
```

## Step 6: Verify

Check the server logs for successful APNs initialization:

```bash
ssh vern "source ~/.nvm/nvm.sh && cd /Users/vern/ai-projects/family-dashboard/server && npx pm2 logs vern-server --lines 20 --nostream 2>&1 | grep -i 'push\|apns'"
```

You should see something like:
```
[Push] 🔔 APNs initialized (development) — Key: ABC123XYZ0, Team: 885845M2A4
```

## What Happens After Setup

Once APNs is configured:

1. **Calendar events approaching** → Push notification hits your lock screen + appears in inbox
2. **Chores assigned** → Push notification + inbox
3. **Pantry items expiring** → Inbox only (future push)
4. **Vern insights** → Inbox only (Phase 3)

The orchestrator respects your preferences (quiet hours, category toggles) and enforces:
- **5-minute minimum** between push notifications
- **Duplicate detection** (same event won't notify twice within 1 hour)
- **7-day auto-cleanup** for dismissed/expired notifications

## Troubleshooting

| Problem | Solution |
|---------|----------|
| No push received | Check `APNS_ENVIRONMENT` matches your build (development for Xcode) |
| "APNs not initialized" in logs | Verify `.p8` file path and Key ID are correct |
| Push works on WiFi but not cellular | Verify Tailscale Funnel is running |
| Badge count wrong | Pull down to refresh in the inbox, or relaunch the app |
