# Outlook & Teams Common Issues

Category: Microsoft 365
Difficulty: Tier 1
Last Updated: May 2026
Environment: Microsoft 365 | Exchange Online | Teams | Windows & Mac | Remote Support

---

### Overview

Outlook and Teams are the two most business-critical applications in a Microsoft 365 environment. When either goes down for a user, productivity stops immediately. This article covers the most common issues encountered in a remote IT support environment and their proven resolutions — organized for fast triage and first contact resolution.

---

### Prerequisites

Before troubleshooting confirm you have:

- Access to Microsoft 365 Admin Center
- Access to Exchange Admin Center (**admin.exchange.microsoft.com**)
- Access to Teams Admin Center (**admin.teams.microsoft.com**)
- Remote access to the user's machine if needed (TeamViewer, Quick Assist, etc.)

---

### Part 1 — Outlook Issues

### Issue 1 — Outlook Not Opening or Crashing on Launch

**Common causes:** Corrupt Outlook profile, conflicting add-ins, recent Windows or Office update

**Steps:**

1. First test — launch Outlook in safe mode:
    - Hold **Ctrl** while clicking the Outlook icon
    - If Outlook opens in safe mode → add-in conflict confirmed
2. If add-in conflict: File → Options → Add-ins → Manage COM Add-ins → Go → uncheck all add-ins → restart Outlook normally → re-enable one at a time to isolate the culprit
3. If safe mode also fails — corrupt profile:
    - Control Panel → Mail → Show Profiles → Add new profile
    - Set new profile as default → re-add the user's Exchange account
    - Test launch with new profile
4. If still failing — repair Office:
    - Settings → Apps → Microsoft 365 → Modify → Quick Repair first
    - If Quick Repair fails → Online Repair
5. Document which fix resolved the issue for future reference

---

### Issue 2 — Outlook Not Syncing or Emails Not Arriving

**Common causes:** Offline mode enabled, OST file corruption, Exchange connectivity issue

**Steps:**

1. Check bottom status bar in Outlook — if it says **"Working Offline"** click Send/Receive → Work Offline to toggle it off
2. Check internet connectivity — open a browser and confirm general access
3. Check Microsoft 365 service health: **admin.microsoft.com** → Health → Service Health → Exchange Online
4. If service health is clear — test connectivity:
    - File → Account Settings → Account Settings → Email tab → double-click the account → Test Account Settings
5. If connectivity test fails — recreate the mail profile (see Issue 1, Step 3)
6. If profile recreation doesn't resolve — delete and rebuild the OST file:
    - Close Outlook
    - Navigate to **C:\Users[username]\AppData\Local\Microsoft\Outlook**
    - Rename the .ost file to .ost.old (do not delete — it's a backup)
    - Reopen Outlook — it will rebuild the OST file automatically
    - Allow time for full mailbox sync depending on mailbox size

---

### Issue 3 — Outlook Password Prompt Keeps Appearing

**Common causes:** Cached credentials outdated, Modern Authentication disabled, MFA change not reflected

**Steps:**

1. Clear cached credentials:
    - Open **Credential Manager** (Windows search)
    - Go to Windows Credentials → find any entries containing "MicrosoftOffice" or "outlook"
    - Remove all of them
    - Restart Outlook → sign in fresh when prompted
2. If prompt returns immediately — check if Modern Authentication is enabled:
    - Navigate to **admin.microsoft.com** → Settings → Org Settings → Modern Authentication
    - Confirm it is enabled — if disabled, enable it and allow up to 24 hours to propagate
3. If user recently changed their password or MFA — credentials need to be re-entered:
    - File → Office Account → Sign Out → Sign back in with updated credentials

---

### Issue 4 — Outlook Autocomplete / Suggested Contacts Not Working

**Steps:**

1. Confirm the autocomplete setting is enabled:
    - File → Options → Mail → Send Messages → check **"Use Auto-Complete List"**
2. If enabled but not working — clear and rebuild the autocomplete cache:
    - File → Options → Mail → Send Messages → **"Empty Auto-Complete List"**
    - Restart Outlook — cache will rebuild as user sends new emails
3. If the issue is that a contact is showing an old email address:
    - Type the contact name → when the suggestion appears, hover over it → click the **X** to remove it
    - The correct address from the directory will populate going forward

---

### Issue 5 — Outlook Calendar Not Syncing or Showing Incorrect Availability

**Steps:**

1. Check if the issue is isolated to one calendar or all calendars
2. For shared calendars: confirm the user has the correct permissions — Calendar Properties → Permissions tab
3. For Free/Busy not showing correctly for other users:
    - Navigate to **Exchange Admin Center** → Recipients → Mailboxes → open affected user
    - Check calendar sharing policy assigned to the mailbox
4. Force a calendar sync: right-click the calendar folder → Sync This Folder
5. If the calendar is completely missing or corrupted — recreate the Outlook profile (see Issue 1, Step 3)

---

### Part 2 — Microsoft Teams Issues

### Issue 1 — Teams Calls Dropping or Poor Audio/Video Quality

**Common causes:** Network congestion, outdated client, driver issues, QoS not configured

**Steps:**

1. Run the **Teams Network Assessment Tool** — download from Microsoft:
    - Tests packet loss, latency, and jitter on the network path Teams uses
    - Acceptable thresholds: packet loss <1%, latency <100ms, jitter <30ms
    - If failing: network issue confirmed — escalate to network team with the report
2. Check Teams client version: click profile picture → About → Version → ensure it's current
3. Update audio/video drivers on the user's device
4. Clear Teams cache:
    - Fully quit Teams (system tray → right-click → Quit)
    - Navigate to **%appdata%\Microsoft\Teams**
    - Delete contents of: Cache, blob_storage, databases, GPUcache, IndexedDB, Local Storage, tmp folders
    - Do not delete the entire Teams folder — only the contents of those subfolders
    - Relaunch Teams
5. Test on Teams web app (**teams.microsoft.com**) — if web works but desktop doesn't, the issue is client-side
6. If issue persists across multiple users — check QoS settings at the network level

---

### Issue 2 — Teams Messages Not Sending or Delayed

**Steps:**

1. Check Microsoft 365 service health — **admin.microsoft.com** → Health → Service Health → Teams
2. If service health is clear — check user's Teams client:
    - Sign out of Teams → sign back in
    - Clear Teams cache (see Issue 1, Step 4)
3. Check if the issue is in a specific channel or chat vs. all of Teams:
    - If specific channel: check channel moderation settings in Teams Admin Center → the user may be restricted from posting
    - If all of Teams: likely an authentication or connectivity issue
4. Test on Teams web app — if messages send on web, reinstall the desktop client

---

### Issue 3 — Teams Not Loading or Stuck on Loading Screen

**Steps:**

1. Check internet connectivity and Microsoft service health first
2. Sign out and back into Teams
3. Clear Teams cache (see Issue 1, Step 4)
4. If still stuck — uninstall and reinstall Teams:
    - Settings → Apps → Teams → Uninstall
    - Also check **C:\Users[username]\AppData\Local\Microsoft\Teams** and delete the folder if it still exists
    - Download fresh installer from **teams.microsoft.com/downloads**
5. Test on Teams web app while desktop is being resolved so user isn't blocked

---

### Issue 4 — Can't Join or Schedule Meetings

**Common causes:** Calendar permissions, Teams meeting policy, license issue

**Steps:**

1. Confirm user has a license with Teams and Exchange Online — both are required for meeting scheduling
2. Check Teams meeting policy in Teams Admin Center:
    - Teams Admin Center → Users → open the user → Policies tab
    - Confirm assigned meeting policy allows scheduling and joining — compare with a working user's policy
3. If the user can't join external meetings:
    - Teams Admin Center → Meetings → Meeting Settings → confirm external access is not blocked
4. If meeting links aren't generating in calendar invites:
    - Confirm the Teams Outlook add-in is installed and enabled
    - Check add-ins in Outlook: File → Options → Add-ins → COM Add-ins → confirm Microsoft Teams Meeting Add-in is checked

---

### Issue 5 — Teams Status Stuck on Away or Showing Incorrect Presence

**Steps:**

1. Have the user sign out and back into Teams — presence refreshes on sign-in
2. Clear Teams cache (see Issue 1, Step 4)
3. If the issue affects how others see the user's presence:
    - This is often a replication delay — allow up to 30 minutes before escalating
4. If presence is consistently wrong regardless of activity:
    - Teams Admin Center → Users → open the user → confirm presence settings under their policy
5. Check if the user has Teams open on multiple devices — presence can conflict across sessions

---

### Part 3 — When to Escalate

Escalate to Tier 2 or Microsoft Support if you observe:

- Issue affects multiple users simultaneously — potential tenant-wide or service outage
- Exchange mailbox corruption confirmed — requires admin-level mailbox repair
- Teams policy changes required at the tenant level
- Service health shows an active incident and no ETA for resolution — communicate status to affected users and set follow-up reminder
- Data loss suspected — emails or messages missing that were previously visible

---

### Part 4 — Quick Reference Triage Checklist

Before diving into deep troubleshooting, run through this fast checklist:

- [ ]  Is Microsoft 365 service health clear?
- [ ]  Is the issue isolated to one user or multiple?
- [ ]  Is the issue on desktop app, web app, or both?
- [ ]  Has anything changed recently — password, MFA, new device, update?
- [ ]  Has the user tried signing out and back in?
- [ ]  Has the user tried the web app as a workaround?

If you can answer all six before your first troubleshooting step, your resolution time drops significantly.

---

### Related Articles

- Microsoft 365 License Management
- MFA Setup, Loss & Recovery
- VPN Connectivity Troubleshooting