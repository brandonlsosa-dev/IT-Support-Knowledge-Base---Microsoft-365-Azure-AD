# Windows Endpoint Diagnostics

Category: Endpoint & Hardware
Difficulty: Tier 2
Last Updated: May 2026
Environment: Windows 10/11 | Remote Support | Intune | Event Viewer | PowerShell

---

### Overview

Endpoint diagnostics is the difference between a technician who restarts computers and hopes for the best, and one who systematically identifies root causes and prevents repeat incidents. This article covers the core diagnostic tools built into Windows — Event Viewer, Resource Monitor, Task Manager, SFC, DISM, and PowerShell — and how to use them together to resolve and document endpoint issues at a tier 2 level.

---

### Prerequisites

Before running diagnostics confirm you have:

- Local administrator rights on the endpoint or remote admin access via Quick Assist, TeamViewer, or RDP
- Access to Intune / Microsoft Endpoint Manager if device is managed
- A ticket open to document all findings and actions taken

---

### Part 1 — Systematic Diagnostic Approach

Never start by randomly applying fixes. Follow this order every time:

`1. Gather — collect symptoms, error messages, recent changes
        ↓
2. Reproduce — confirm you can see the issue yourself
        ↓
3. Isolate — narrow down to hardware, software, OS, or network
        ↓
4. Identify — use diagnostic tools to find the root cause
        ↓
5. Resolve — apply the targeted fix
        ↓
6. Verify — confirm the issue is fully resolved
        ↓
7. Document — record root cause and resolution in the ticket`

Skipping steps 1–3 leads to wasted time applying fixes to the wrong layer of the problem.

---

### Part 2 — Task Manager

Task Manager is your fastest first look at endpoint health. Open with **Ctrl + Shift + Esc.**

### Performance Tab

Gives you a real-time view of system resource usage:

| Metric | What to Look For |
| --- | --- |
| CPU | Sustained usage above 90% — identify the process causing it |
| Memory | Usage above 85% of total RAM — check for memory leaks or insufficient RAM |
| Disk | Sustained 100% disk usage — common cause of slow performance on HDD systems |
| GPU | Unexpected high GPU usage on a non-graphics workload |
| Ethernet/WiFi | Unusual network activity when the user isn't doing anything |

### Processes Tab

1. Click **"CPU"** column header to sort by CPU usage — highest consumers at top
2. Click **"Memory"** column to sort by RAM usage
3. Right-click any suspicious process → **"Open file location"** to verify legitimacy
4. Right-click → **"Search online"** to research unknown processes
5. Never end a process you can't identify — research first

### Startup Tab

Controls what launches at Windows startup:

1. Review all enabled startup items
2. Right-click unnecessary items → **Disable**
3. Common performance killers: Teams, OneDrive, Spotify, browser apps set to auto-launch
4. Disabling startup items does not uninstall them — it just stops them from launching automatically

---

### Part 3 — Event Viewer

Event Viewer is the most powerful built-in diagnostic tool in Windows. Every crash, error, and warning is logged here with timestamps, error codes, and source information.

### How to Open

- Windows search → **"Event Viewer"** → run as administrator

### Key Log Locations

| Log | What It Contains |
| --- | --- |
| Windows Logs → System | OS-level errors, driver failures, service crashes |
| Windows Logs → Application | App crashes, software errors, .NET failures |
| Windows Logs → Security | Login events, account lockouts, audit logs |
| Applications and Services → Microsoft → Windows → Diagnostics-Performance | Boot and shutdown performance issues |

### How to Read an Event

Click any event to see:

- **Level:** Information, Warning, Error, or Critical
- **Date and Time:** exact timestamp of the event
- **Source:** which component or application generated it
- **Event ID:** the specific error code — use this to research the exact issue
- **Description:** human-readable explanation of what happened

### Filtering Events — The Right Way

Don't scroll through thousands of events manually:

1. Right-click the log → **"Filter Current Log"**
2. Filter by:
    - **Event level:** check Error and Critical only
    - **Event sources:** filter to a specific app or driver
    - **Time range:** narrow to when the issue occurred
3. This surfaces only relevant events immediately

### Most Useful Event IDs to Know

| Event ID | Log | Meaning |
| --- | --- | --- |
| 41 | System | Kernel power failure — unexpected shutdown or crash |
| 1001 | Application | Windows Error Reporting — application crash details |
| 6008 | System | Unexpected shutdown — system didn't shut down cleanly |
| 7034 | System | Service crashed unexpectedly |
| 4625 | Security | Failed login attempt |
| 4740 | Security | Account lockout — shows which machine triggered it |
| 1074 | System | System was restarted or shut down intentionally and by whom |

### Using Event ID 4740 to Find Account Lockout Source

This is one of the most practical Event Viewer skills in IT support:

1. Open Event Viewer on the **Domain Controller** (or check Azure AD Sign-In Logs for cloud accounts)
2. Filter Security log for Event ID 4740
3. The event details show:
    - Which account was locked
    - Which machine triggered the lockout
    - Timestamp of lockout
4. Go to the triggering machine and check for saved credentials, mapped drives, or scheduled tasks using old credentials — these are the most common lockout causes

---

### Part 4 — Resource Monitor

Resource Monitor gives deeper detail than Task Manager for CPU, memory, disk, and network activity.

### How to Open

Task Manager → Performance tab → **"Open Resource Monitor"** at the bottom

### CPU Tab

- Shows per-process CPU usage with more detail than Task Manager
- **"Associated Handles"** section: search for a file or process name to see which apps have it open — useful when a file can't be deleted because something is using it

### Disk Tab

- Shows real-time read/write activity per process
- Look for processes with sustained high disk I/O when performance is degraded
- **100% disk usage in Task Manager + high I/O in Resource Monitor on a specific process** = root cause identified

### Network Tab

- Shows per-process network activity
- Look for unexpected outbound connections from unknown processes — potential malware indicator
- Note the remote addresses and research any you don't recognize

### Memory Tab

- Shows physical memory usage per process
- **"Hard Faults/sec"** column: high numbers indicate the system is using the page file heavily — RAM is insufficient for current workload

---

### Part 5 — SFC and DISM

Use these tools when Windows is behaving erratically, crashing, or showing errors that don't trace back to a specific application — likely OS file corruption.

### SFC — System File Checker

Scans and repairs corrupted Windows system files.

**Run as administrator in Command Prompt:**

`sfc /scannow`

- Scan takes 10–20 minutes
- Results:
    - **"Windows Resource Protection did not find any integrity violations"** — OS files are clean
    - **"Windows Resource Protection found corrupt files and repaired them"** — corruption found and fixed, restart required
    - **"Windows Resource Protection found corrupt files but was unable to fix some of them"** — run DISM next

---

### DISM — Deployment Image Servicing and Management

Repairs the Windows component store that SFC relies on. Run DISM if SFC couldn't fix all files.

**Run as administrator in Command Prompt — run in this order:**

`DISM /Online /Cleanup-Image /CheckHealth`

Checks for known corruption without making changes — fast, run first.

`DISM /Online /Cleanup-Image /ScanHealth`

Full scan for corruption — takes longer, more thorough.

`DISM /Online /Cleanup-Image /RestoreHealth`

Downloads and replaces corrupted components from Windows Update — requires internet connection.

**After DISM RestoreHealth completes — run SFC again:**

`sfc /scannow`

SFC can now repair files it previously couldn't using the restored component store.

**After both complete — restart the device and test.**

---

### Part 6 — PowerShell Diagnostic Commands

These commands give you fast answers that would take multiple clicks in the GUI.

### System Information

powershell

`# Full system info summary
Get-ComputerInfo | Select-Object CsName, WindowsVersion, OsArchitecture, CsTotalPhysicalMemory

# Check uptime — long uptime can cause performance issues
(Get-Date) - (gcim Win32_OperatingSystem).LastBootUpTime

# List installed software
Get-WmiObject -Class Win32_Product | Select-Object Name, Version | Sort-Object Name`

### Disk Health

powershell

`# Check disk health status
Get-PhysicalDisk | Select-Object FriendlyName, MediaType, HealthStatus, OperationalStatus

# Check disk space on all drives
Get-PSDrive -PSProvider FileSystem | Select-Object Name, Used, Free`

### Network Diagnostics

powershell

`# Test connectivity to a specific address
Test-NetConnection -ComputerName google.com -Port 443

# Check DNS resolution
Resolve-DnsName google.com

# View current network adapters and IP addresses
Get-NetIPAddress | Select-Object InterfaceAlias, IPAddress, AddressFamily`

### Services

powershell

`# Find stopped services that should be running
Get-Service | Where-Object {$_.StartType -eq 'Automatic' -and $_.Status -ne 'Running'}

# Restart a specific service
Restart-Service -Name "Spooler" -Force`

### Event Log Query via PowerShell

powershell

`# Pull last 20 system errors
Get-EventLog -LogName System -EntryType Error -Newest 20 | Select-Object TimeGenerated, Source, Message

# Find account lockout events
Get-EventLog -LogName Security -InstanceId 4740 -Newest 10`

---

### Part 7 — Performance Troubleshooting Workflow

When a user reports a slow computer, follow this sequence:

**Step 1 — Task Manager quick check (2 min)**

- Is CPU, RAM, or Disk at sustained high usage?
- Identify the top consuming process

**Step 2 — Startup items (5 min)**

- Task Manager → Startup tab
- Disable unnecessary items
- Restart and test

**Step 3 — Check disk health (5 min)**

powershell

`Get-PhysicalDisk | Select-Object FriendlyName, MediaType, HealthStatus`

- If HDD and not SSD — disk speed is likely the bottleneck. Flag for hardware upgrade.
- If HealthStatus shows Warning or Unhealthy — escalate immediately, potential data loss risk

**Step 4 — Check uptime**

powershell

`(Get-Date) - (gcim Win32_OperatingSystem).LastBootUpTime`

- If uptime is 30+ days — a restart alone may resolve the issue
- Schedule a restart during off-hours if the user can't restart immediately

**Step 5 — Run SFC if OS-level issues suspected**

- If errors trace back to Windows components rather than a specific app

**Step 6 — Check Event Viewer**

- Filter System and Application logs for Errors and Criticals
- Look for patterns — same error repeating at the same time daily often indicates a scheduled task or service conflict

---

### Part 8 — When to Escalate

Escalate to tier 3 or vendor support if:

- SMART data indicates imminent hard drive failure — data backup and hardware replacement required urgently
- Blue Screen of Death (BSOD) occurring repeatedly — collect minidump files from **C:\Windows\Minidump** and escalate with dumps attached
- SFC and DISM cannot repair corruption — OS reinstall may be required
- Malware suspected based on unusual network activity or unknown processes — escalate to security team immediately, isolate the device from the network
- Hardware diagnostics indicate failing RAM, GPU, or motherboard — escalate to hardware team or vendor

---

### Part 9 — Documentation Standard for Endpoint Tickets

Every endpoint ticket should close with:

- **Reported symptom:** what the user described
- **Reproduced:** yes or no — and how
- **Root cause identified:** specific cause found via diagnostics
- **Tools used:** Event Viewer / Task Manager / SFC / PowerShell / etc.
- **Resolution applied:** exact steps taken
- **Verified by:** confirmed working by user or self-test
- **Recurrence prevention:** any changes made to prevent the issue returning

---

### Related Articles

- VPN Connectivity Troubleshooting
- Outlook & Teams Common Issues
- Password Reset & Account Unlock Procedures