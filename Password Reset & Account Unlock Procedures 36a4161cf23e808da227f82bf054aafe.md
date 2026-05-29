# Password Reset & Account Unlock Procedures

Category: Identity & Access
Difficulty: Tier 1
Last Updated: May 2026
Environment: Active Directory | Azure AD / Entra ID | Microsoft 365 | Remote Support

---

### Overview

Password resets and account unlocks are the highest volume ticket category in virtually every IT support environment. While the individual tasks are straightforward, the security protocols surrounding them are not. Improper identity verification before a reset is one of the most exploited social engineering vectors in IT — attackers frequently call help desks pretending to be employees to gain unauthorized access.

This article covers the full procedure for both on-premises Active Directory and cloud-based Azure AD environments, with security protocol built into every step.

---

### Prerequisites

Before performing any reset or unlock confirm you have:

- Help Desk Administrator, User Administrator, or equivalent role
- Access to Active Directory Users and Computers (on-premises) and/or Azure AD portal
- A clear identity verification method confirmed before making any changes
- An open ticket to document every action taken

---

### Part 1 — Identity Verification Protocol

This step is non-negotiable and must be completed before every single reset or unlock regardless of how well you think you know the caller.

### Verification Methods — Require at Least Two

| Method | How to Verify |
| --- | --- |
| Employee ID | Ask user to provide it — cross-check against HR system or AD attribute |
| Manager confirmation | Call or message the user's manager directly to confirm the request |
| HR record match | Confirm start date, department, or job title against HR records |
| Security question | If configured in your environment's self-service portal |
| Video verification | For high-security environments — confirm via Teams or Zoom with camera on |

### Red Flags — Do Not Proceed If:

- Caller cannot provide two verification factors
- Caller is unusually urgent or pressuring you to skip verification
- Request comes in outside business hours without prior notice
- Caller claims to be an executive but can't verify basic identity details — CEO fraud is a real and common attack vector
- Request is for a shared or service account rather than a personal user account — escalate these

> ⚠️ **Security Note:** Document your verification method in every ticket. If a security incident occurs later involving a reset you performed, your documentation is your protection.
> 

---

### Part 2 — Active Directory Password Reset (On-Premises)

### Step 1 — Open Active Directory Users and Computers

1. On a domain controller or management workstation → open **Active Directory Users and Computers (ADUC)**
2. Navigate to the correct Organizational Unit (OU) for the user
3. Or use **Find** (Ctrl + F) to search by name or username directly

### Step 2 — Check Account Status First

Before resetting — right-click the user → **Properties → Account tab:**

- Is the account locked? → checkbox "Unlock account" will be available if so
- Is the account disabled? → confirm with your manager before re-enabling
- Has the account expired? → check the account expiration date field
- Note the last logon date — if it's very old, confirm the user is still active with HR

### Step 3 — Unlock the Account (If Locked)

1. Right-click the user → **Properties → Account tab**
2. Check **"Unlock account"** checkbox
3. Click **Apply**
4. Do not reset the password yet unless the user has also forgotten it — unlocking alone restores access if the password is known

### Step 4 — Reset the Password (If Required)

1. Right-click the user → **"Reset Password"**
2. Enter a temporary password that meets your organization's complexity requirements:
    - Minimum 12 characters recommended
    - Must include uppercase, lowercase, number, and special character
    - Never use predictable patterns like Welcome1! or Company2026!
3. Check **"User must change password at next logon"** — always enable this
4. Uncheck **"User cannot change password"** — confirm this is not checked
5. Click **OK**
6. Communicate the temporary password to the user via a secure channel — never email it in plain text

### Step 5 — Verify

1. Confirm with the user that they can log in with the temporary password
2. Confirm the password change prompt appears on login
3. Confirm they successfully set a new password
4. Test access to key resources — workstation login, email, shared drives

### Step 6 — Document and Close

- Identity verification method used ✅
- Account locked: yes/no ✅
- Account unlocked: yes/no ✅
- Password reset: yes/no ✅
- Temporary password communicated via: [method] ✅
- User confirmed access restored ✅
- Completed by: [your name] | Date: [date]

---

### Part 3 — Azure AD Password Reset (Cloud)

### Via Microsoft 365 Admin Center

1. Navigate to **admin.microsoft.com** → Users → Active Users
2. Search for and open the user
3. Click **"Reset password"** in the top action bar
4. Choose:
    - **Auto-generate password** — Microsoft generates a complex temporary password
    - **Let me create the password** — you set it manually
5. Check **"Require this user to change their password when they first sign in"** — always enable
6. Click **"Reset password"**
7. Copy the temporary password — you will only see it once
8. Communicate to the user securely

### Via Azure AD Portal

1. Navigate to **portal.azure.com** → Azure Active Directory → Users
2. Search for and open the user
3. Click **"Reset password"** in the top menu
4. Follow the same steps as above

### Checking for Azure AD Account Lockout

Azure AD Smart Lockout automatically locks accounts after repeated failed attempts. Unlike on-premises AD there is no manual unlock button — the lockout clears automatically after the lockout duration expires (typically 60 seconds to several minutes depending on policy).

**What you can do:**

1. Navigate to Azure AD → Users → open the user
2. Check **"Sign-in logs"** for failed attempts and lockout events
3. Confirm the account is not disabled — that requires manual re-enabling
4. Advise the user to wait 5 minutes then try again with the correct credentials
5. If lockout persists — reset the password which clears the lockout state immediately

---

### Part 4 — Self-Service Password Reset (SSPR)

If your organization has Azure AD Self-Service Password Reset enabled, users can reset their own passwords without contacting IT. This dramatically reduces ticket volume.

### How to Confirm SSPR is Enabled

1. Navigate to **portal.azure.com** → Azure Active Directory → Password Reset
2. Confirm **"Self-service password reset enabled"** is set to All or Selected
3. Check the **"Authentication methods"** tab — confirm at least 2 methods are required:
    - Mobile app notification
    - Mobile app code
    - Email
    - Mobile phone
    - Security questions

### Directing Users to SSPR

Send users to: **aka.ms/sspr**

They will be prompted to verify their identity using their registered methods and can set a new password immediately without IT involvement.

### When SSPR Fails

Users may be unable to use SSPR if:

- They never registered authentication methods — redirect to **aka.ms/ssprsetup** to register
- Their registered phone number or email is outdated — update in Azure AD → User → Authentication Methods
- SSPR is not licensed — requires Azure AD P1 or P2, Microsoft 365 Business Premium, or E3/E5

---

### Part 5 — Service Account and Shared Account Resets

These require elevated caution and should never be handled the same as standard user resets.

### Before Resetting a Service Account Password:

1. Identify every system and application using that service account — check with your systems admin
2. Schedule the reset during a maintenance window — resetting during business hours can break services immediately
3. After reset — update the password in every dependent system:
    - Windows Services using the account (services.msc → right-click service → Properties → Log On tab)
    - Scheduled Tasks using the account
    - IIS application pools
    - Any third-party applications configured with the account
4. Test all dependent services after the update
5. Document every system that was updated

> ⚠️ **Never reset a service account password without knowing every system that uses it. This is one of the most common causes of widespread outages in IT environments.**
> 

---

### Part 6 — Communicating Temporary Passwords Securely

Never send a temporary password in plain text email or Teams message. Use one of these methods instead:

| Method | When to Use |
| --- | --- |
| Phone call | Most secure — verbal communication only, no written record |
| Encrypted email | If your org uses Microsoft Purview Message Encryption or S/MIME |
| IT portal / self-service | Best practice — system generates and delivers password automatically |
| In person | For on-site users when possible |
| One-time secret tool | Tools like onetimesecret.com generate a link that expires after one view |

**Never use:**

- Standard Teams or Slack message — logs are retained and searchable
- Plain text email — stored in both sender and recipient mailboxes
- Ticket description field — often visible to multiple agents and logged indefinitely

---

### Part 7 — Password Policy Reference

Document your organization's password policy here for quick reference during resets:

| Requirement | Standard Recommendation |
| --- | --- |
| Minimum length | 12 characters |
| Complexity | Uppercase, lowercase, number, special character |
| Expiration | 90 days or never (NIST now recommends no expiration unless compromised) |
| History | Cannot reuse last 10 passwords |
| Lockout threshold | 5–10 failed attempts |
| Lockout duration | 15–30 minutes or until admin unlock |
| SSPR | Enabled for all users with 2 methods required |

---

### Part 8 — Common Issues & Troubleshooting

**User resets password but still can't log in**
→ Check if the account is still locked after the reset. Reset and unlock are separate actions in on-premises AD — confirm both were performed.

**Password reset prompt not appearing after reset**
→ Confirm "User must change password at next logon" was checked. If the user is logging into a cached session — have them sign out fully and sign back in.

**User says their new password isn't working**
→ Confirm they are not typing the old password from muscle memory. Have them type it slowly. Also confirm Caps Lock is off. If still failing — reset again and have them type the temporary password directly from your communication.

**SSPR registration prompt not appearing**
→ Check Conditional Access policies — a policy may be blocking the SSPR registration endpoint. Confirm aka.ms/ssprsetup is accessible from the user's location.

**Account keeps getting locked repeatedly**
→ Do not just keep unlocking — find the source. Use Event ID 4740 in Event Viewer on the domain controller to identify which machine is triggering the lockout. Common causes: old saved credentials in Windows Credential Manager, a mapped drive using old credentials, a mobile device with an old password still trying to sync email.

**User locked out of SSPR as well**
→ SSPR itself has rate limiting — too many failed SSPR attempts will temporarily block the user. Perform a manual reset and advise the user to wait before attempting SSPR again.

---

### Part 9 — Quick Reference

| Scenario | Action |
| --- | --- |
| Account locked, password known | Unlock only — do not reset |
| Account locked, password forgotten | Unlock AND reset |
| Password forgotten, not locked | Reset only |
| Account disabled | Confirm with manager before re-enabling |
| Service account reset | Schedule maintenance window, update all dependent systems |
| Suspicious reset request | Refuse, document, escalate to security team |

---

### Related Articles

- Azure AD User Provisioning & Offboarding
- MFA Setup, Loss & Recovery
- Conditional Access Policy Troubleshooting