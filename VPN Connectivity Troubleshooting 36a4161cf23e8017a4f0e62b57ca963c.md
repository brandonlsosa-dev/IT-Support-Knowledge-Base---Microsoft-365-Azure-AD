# VPN Connectivity Troubleshooting

Category: Network & Connectivity
Difficulty: Tier 1-2
Last Updated: May 2026
Environment: Remote Support | Windows & Mac | Cisco AnyConnect | Microsoft Always On VPN

---

### Overview

VPN connectivity is the lifeline of every remote worker. When VPN goes down, the user loses access to every internal resource — file shares, internal apps, printers, and in some cases even email. For remote IT support roles, VPN troubleshooting is one of the highest volume and most time-sensitive ticket categories you will handle.

This article covers end-to-end VPN troubleshooting for the two most common enterprise VPN clients: Cisco AnyConnect and Microsoft Always On VPN.

---

### Prerequisites

Before troubleshooting confirm you have:

- Remote access to the user's machine if needed (Quick Assist, TeamViewer, etc.)
- VPN server address and organization's VPN gateway information
- Access to Azure AD / Active Directory to check account status
- Basic network diagnostic tools available (ping, tracert, nslookup)

---

### Part 1 — First Response Triage

Before touching any VPN settings run through this checklist. It resolves the majority of VPN tickets in under 5 minutes:

- [ ]  Is the user's internet connection working? (Have them open a browser and go to google.com)
- [ ]  Is the VPN client installed and up to date?
- [ ]  Has the user's password recently expired or been changed?
- [ ]  Is the user's account locked in Active Directory or Azure AD?
- [ ]  Is the VPN server itself down? (Check with another user or your network team)
- [ ]  Has anything changed on the user's device recently — update, new network, new router?

**If account is locked or password expired → resolve that first. VPN authentication will fail regardless of any other fix until the account issue is resolved.**

---

### Part 2 — Cisco AnyConnect Troubleshooting

Cisco AnyConnect is the most widely deployed enterprise VPN client. These are the most common errors and their resolutions.

---

### Error — "Authentication Failed"

**Common causes:** Wrong password, expired password, locked AD account, MFA sync issue

**Steps:**

1. Confirm the user is entering their current network password — not a cached or old one
2. Check AD/Azure AD account status:
    - Active Directory Users and Computers → find user → confirm account is not locked
    - Azure AD → Users → confirm account is enabled and not locked
3. Unlock account if locked and have user attempt VPN again
4. If password recently changed — AnyConnect may be caching old credentials:
    - Open AnyConnect → click the gear icon → Preferences → uncheck "Remember my credentials" → reconnect and enter new credentials fresh
5. If MFA is required for VPN:
    - Confirm user's authenticator app is time-synced (see MFA article)
    - Have user approve the MFA prompt immediately after entering credentials — AnyConnect times out quickly

---

### Error — "Unable to Contact Server" or "Connection Attempt Failed"

**Common causes:** No internet, firewall blocking VPN ports, VPN server down, split DNS issue

**Steps:**

1. Confirm internet is working — browser test
2. Check if VPN server is reachable:
    - Open Command Prompt → type `ping [VPN server address]`
    - If no response — server may be down or ports are being blocked
3. Check firewall — AnyConnect requires UDP port 443 and 500 to be open:
    - Temporarily disable Windows Defender Firewall → test VPN connection
    - If VPN connects with firewall off — firewall rule is blocking VPN traffic
    - Re-enable firewall → add exception for AnyConnect
4. Check if the user is on a network that blocks VPN (hotel, public WiFi, some ISPs):
    - Have user try connecting from a different network — mobile hotspot is a reliable test
    - If hotspot works but home network doesn't — ISP or router is blocking VPN traffic
    - Solution: enable **SSL/TLS mode** in AnyConnect if available, or have user contact their ISP
5. If server appears down — escalate to network team immediately and communicate status to user

---

### Error — "VPN Service Not Available" or AnyConnect Won't Launch

**Common causes:** VPN service stopped, corrupt installation, Windows update conflict

**Steps:**

1. Restart the VPN service:
    - Press **Windows + R** → type `services.msc` → Enter
    - Find **"Cisco AnyConnect VPN Agent"**
    - Right-click → Restart
    - Try launching AnyConnect again
2. If service won't start — check Windows Event Viewer for errors:
    - Event Viewer → Windows Logs → Application → filter by "AnyConnect"
    - Note the error code and look up in Cisco's documentation
3. If service issue persists — reinstall AnyConnect:
    - Uninstall via Settings → Apps
    - Also remove leftover files in **C:\ProgramData\Cisco\Cisco AnyConnect Secure Mobility Client**
    - Reinstall from your organization's software deployment tool or IT portal
4. After reinstall — confirm the VPN service is running before testing connection

---

### Error — Connected to VPN But Can't Access Internal Resources

**Common causes:** Split tunnel configuration, DNS not resolving internal addresses, routing issue

**Steps:**

1. Confirm the VPN shows as connected in AnyConnect
2. Test internal resource access by IP address instead of hostname:
    - If IP access works but hostname doesn't → DNS issue
    - Open Command Prompt → `nslookup [internal resource name]`
    - If DNS resolution fails → VPN is not pushing the internal DNS server to the client
3. Check DNS settings while connected to VPN:
    - Command Prompt → `ipconfig /all` → look for DNS servers listed
    - Internal DNS server should appear — if it doesn't, escalate to network team
4. Flush DNS cache and retry:
    - Command Prompt (as admin) → `ipconfig /flushdns`
    - Try accessing internal resource again
5. If IP access also fails — routing issue:
    - Command Prompt → `tracert [internal resource IP]`
    - Share the tracert output with your network team for analysis

---

### Part 3 — Microsoft Always On VPN Troubleshooting

Always On VPN connects automatically when the device starts, without user interaction. Issues here are less frequent but more complex when they occur.

---

### Issue — Always On VPN Not Connecting Automatically

**Steps:**

1. Check if the device is domain-joined or Azure AD joined — Always On VPN requires one of these
2. Check VPN profile deployment:
    - Open Settings → Network & Internet → VPN → confirm the VPN profile is present
    - If missing — profile was not deployed via Intune or Group Policy correctly
    - Escalate to your MDM/Intune admin to redeploy the configuration profile
3. Check the device tunnel vs. user tunnel:
    - Device tunnel connects before user login — used for domain connectivity
    - User tunnel connects after login — used for resource access
    - Confirm which tunnel is failing before troubleshooting further
4. Review event logs:
    - Event Viewer → Applications and Services Logs → Microsoft → Windows → VPN → Operational
    - Look for connection failure events and note the error codes

---

### Issue — Always On VPN Dropping Intermittently

**Steps:**

1. Check network stability — intermittent drops often follow the underlying connection
2. Review VPN event logs for disconnect reason codes
3. Check if the issue correlates with the device going to sleep:
    - Power settings may be suspending the network adapter
    - Device Manager → Network Adapters → right-click the adapter → Properties → Power Management → uncheck "Allow the computer to turn off this device to save power"
4. If drops are random with no sleep correlation — escalate to network team with event log exports

---

### Part 4 — Mac VPN Troubleshooting

Most steps mirror Windows but with Mac-specific paths.

### AnyConnect on Mac

**VPN won't launch:**

1. Check System Preferences → Security & Privacy → confirm AnyConnect is allowed under "Allow apps downloaded from"
2. Restart the AnyConnect service via Terminal:

`sudo launchctl stop com.cisco.anyconnect.vpnagentd
sudo launchctl start com.cisco.anyconnect.vpnagentd`

1. If still failing — reinstall AnyConnect from your IT portal

**Connected but no internal access:**

1. Open Terminal → `scutil --dns` → confirm internal DNS servers are listed while VPN is connected
2. Flush DNS cache:

`sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder`

1. Test internal resource access again

---

### Part 5 — Collecting Diagnostic Information for Escalation

If VPN issues can't be resolved at tier 1-2, collect the following before escalating:

**From the user's machine:**

- AnyConnect logs: Launch AnyConnect → gear icon → **"Export Logs"** → save the zip file
- Windows Event Viewer export: filter for AnyConnect or VPN entries → right-click → Save All Events
- Output of `ipconfig /all` while VPN is connected and while disconnected
- Output of `tracert [VPN server address]`

**From your admin tools:**

- Azure AD Sign-In Logs for the user around the time of failure
- Any recent changes to VPN gateway, firewall rules, or Conditional Access policies

**Always document:** exact error message, time of failure, network the user was on, and what changed recently. This cuts escalation resolution time significantly.

---

### Part 6 — Common Quick Fixes Reference

| Symptom | First Fix to Try |
| --- | --- |
| Authentication Failed | Check for account lockout in AD/Azure AD |
| Can't reach server | Test on mobile hotspot to isolate network |
| Connected but no access | Run `ipconfig /flushdns` and test by IP |
| AnyConnect won't launch | Restart VPN service in services.msc |
| Always On not connecting | Check if VPN profile is present in Settings |
| Mac VPN issues | Check Security & Privacy app permissions |
| Drops intermittently | Check power management on network adapter |

---

### Related Articles

- MFA Setup, Loss & Recovery
- Windows Endpoint Diagnostics
- Outlook & Teams Common Issues