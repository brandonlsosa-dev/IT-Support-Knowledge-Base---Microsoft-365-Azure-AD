# Conditional Access Policy Troubleshooting

Category: Identity & Access
Difficulty: Tier 3
Last Updated: May 2026
Environment: Microsoft 365 / Entra ID | Azure AD | Remote Support

---

### Overview

Conditional Access is Azure AD's policy engine that controls how and when users can access company resources. It evaluates signals — user identity, device compliance, location, app being accessed — and either grants access, blocks it, or requires additional verification like MFA.

When Conditional Access breaks, users get locked out with cryptic error codes and no clear path to resolution. This article covers how to diagnose, troubleshoot, and resolve Conditional Access issues without creating security gaps in the process.

---

### Prerequisites

Before troubleshooting any Conditional Access issue, confirm you have:

- Security Administrator or Conditional Access Administrator role in Azure AD
- Access to Azure AD Sign-In Logs
- Access to the Conditional Access policy blade in Entra ID
- The user's exact error message and sign-in timestamp

---

### Part 1 — Understanding How Conditional Access Works

Before troubleshooting, understand the evaluation order:

`User attempts sign-in
        ↓
Azure AD evaluates ALL Conditional Access policies
        ↓
If ANY policy blocks → Access Denied
If ALL policies pass → Access Granted (with any required controls applied)
        ↓
Controls applied: MFA, compliant device, approved app, etc.`

**Key principle:** Policies are evaluated simultaneously, not sequentially. A user can be granted by one policy and blocked by another — the block always wins.

---

### Part 2 — Reading the Sign-In Logs

The Sign-In Log is your first stop for every Conditional Access issue.

### How to Access

1. Navigate to **portal.azure.com** → Azure Active Directory → Sign-In Logs
2. Filter by:
    - **User:** the affected user's name or UPN
    - **Date:** narrow to the time the issue occurred
    - **Status:** Failed or Interrupted

### What to Look For

Click on the failed sign-in event → go to the **"Conditional Access"** tab

You will see every policy that was evaluated and its result:

| Result | Meaning |
| --- | --- |
| Success | Policy evaluated and access granted |
| Failure | Policy evaluated and access blocked |
| Not Applied | User or app not in policy scope |
| Report Only | Policy in audit mode, not enforcing |

**Find the policy showing "Failure" — that is your culprit.**

### Common Failure Reasons

- **"Device is not compliant"** — device not enrolled in Intune or failing compliance policy
- **"MFA required but not satisfied"** — user didn't complete MFA or has no method registered
- **"Access blocked by policy"** — explicit block policy is applying to this user
- **"Location not allowed"** — user is signing in from a blocked country or IP range
- **"Application not approved"** — user is trying to access an app not covered by policy

---

### Part 3 — Common Scenarios & Fixes

### Scenario 1 — User Blocked After New Policy Deployment

**Symptom:** User was working fine, then suddenly blocked after an admin made changes.

**Steps:**

1. Pull Sign-In Logs → identify which new policy is blocking
2. Navigate to **Azure AD → Security → Conditional Access → Policies**
3. Open the new policy → check **Users and Groups** assignment
4. Confirm whether the user should be in scope or was accidentally included
5. If accidental: remove the user or their group from the policy assignment
6. If intentional: determine if the user needs an exception → add to **Exclusions** with documented business justification
7. Test sign-in after change → confirm access restored

---

### Scenario 2 — "Device Not Compliant" Error

**Symptom:** User gets blocked with device compliance error, especially after getting a new device or after a policy change.

**Steps:**

1. Confirm the device is enrolled in **Intune**: Microsoft Endpoint Manager → Devices → search device name
2. If not enrolled: walk user through enrollment
    - Windows: Settings → Accounts → Access Work or School → Connect
    - Mac: Install Company Portal app → enroll device
3. If enrolled but showing non-compliant: check **Intune → Devices → [Device Name] → Device Compliance**
4. Identify which compliance requirement is failing:
    - BitLocker not enabled
    - OS version out of date
    - Antivirus not reporting
    - PIN/password not set
5. Remediate the failing requirement on the device
6. Force a compliance check sync: Intune portal → device → Sync
7. Wait 15–30 minutes for compliance status to update in Azure AD
8. Test sign-in → confirm access restored

---

### Scenario 3 — MFA Loop (User Prompted for MFA Repeatedly)

**Symptom:** User completes MFA but keeps getting prompted again on every sign-in or every few minutes.

**Steps:**

1. Pull Sign-In Logs → check if "Sign-in frequency" control is applied
2. Navigate to the policy → **Session Controls → Sign-in Frequency**
3. Confirm whether the frequency is intentional (security requirement) or misconfigured
4. If misconfigured: adjust the frequency or enable **"Persistent browser session"** for low-risk users
5. Also check: is the user clearing browser cookies between sessions? Persistent session tokens are stored in cookies — clearing them forces re-authentication
6. Confirm whether "Remember this device" is conflicting with a sign-in frequency override

---

### Scenario 4 — Policy Applies to Wrong Users

**Symptom:** Users who should not be affected by a policy are getting blocked.

**Steps:**

1. Navigate to **Conditional Access → [Policy] → Users and Groups**
2. Review the **Include** section — check for "All Users" being used unintentionally
3. Review the **Exclude** section — confirm break-glass accounts and service accounts are excluded
4. Use the **"What If"** tool to test policy impact before making changes:
    - Conditional Access → What If
    - Enter the affected user, app, device, and location
    - See exactly which policies apply and why
5. Adjust Include/Exclude as needed → document all changes in your ticketing system

---

### Scenario 5 — Block Policy Affecting Guest or External Users

**Symptom:** External partner or vendor can't access shared resources after a policy change.

**Steps:**

1. Pull Sign-In Logs for the guest account (search by their external email)
2. Identify the blocking policy → check if Guest users are in scope
3. Navigate to the policy → **Users and Groups → Include**
4. If "All Users" is selected, it includes guests — this is a common oversight
5. Options to resolve:
    - Create a separate Conditional Access policy scoped specifically to Guest users
    - Add the guest user's domain to an exclusion group
    - Use **Cross-Tenant Access Settings** to manage guest access more granularly
6. Test external sign-in after change → confirm access

---

### Part 4 — Using the What If Tool

The What If tool lets you simulate policy evaluation before deploying changes. Use it before every policy modification.

### How to Use

1. Navigate to **Azure AD → Security → Conditional Access → What If**
2. Enter:
    - **User:** the user you want to test
    - **Cloud app:** the app they're trying to access
    - **IP address:** their location if relevant
    - **Device platform:** Windows, iOS, Android, etc.
    - **Device state:** compliant or not
3. Click **"What If"**
4. Review every policy that would apply and what action it would take

**Use this before and after every policy change.** It takes 2 minutes and prevents lockouts.

---

### Part 5 — Break-Glass Accounts

Every environment should have at least 2 break-glass accounts — emergency admin accounts excluded from all Conditional Access policies. If you lock out all admins via a bad policy, break-glass accounts are your recovery path.

**Verify break-glass accounts are configured:**

1. Navigate to **Conditional Access → each policy → Exclusions**
2. Confirm break-glass accounts or group are excluded from every policy
3. Break-glass accounts should have:
    - Strong random password stored securely offline
    - No MFA requirement (the point is emergency access)
    - Monitored via alerts — any sign-in should trigger an immediate alert
    - Never used for day-to-day admin tasks

---

### Part 6 — Escalation Triggers

Escalate immediately if you observe any of the following:

- A Conditional Access policy was modified without a change ticket
- Sign-In Logs show a break-glass account was used
- A user is being blocked from a location they've never signed in from before — potential compromise
- Multiple users locked out simultaneously after a policy change — rollback immediately
- Policy changes made outside business hours without approval

---

### Related Articles

- Azure AD User Provisioning & Offboarding
- MFA Setup, Loss & Recovery
- Microsoft 365 License Management