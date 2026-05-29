# Azure AD User Provisioning & Offboarding

Category: Identity & Access
Difficulty: Tier 2
Last Updated: May 2026
Environment: Microsoft 365 / Entra ID | Windows Endpoints | Remote Support

---

### Overview

User provisioning and offboarding are among the most security-critical workflows in IT support. Improper provisioning leads to access gaps that block productivity. Improper offboarding creates security vulnerabilities including unauthorized access to company data, active licenses on inactive accounts, and unrevoked third-party app permissions.

This article documents the end-to-end process for both workflows in a Microsoft 365 / Azure AD environment.

---

### Prerequisites

Before starting either workflow, confirm you have:

- Global Administrator or User Administrator role in Azure AD
- Access to Microsoft 365 Admin Center
- Access to Azure AD / Entra ID portal
- Access to your organization's HR system or ticketing platform for identity verification

---

### Part 1 — User Provisioning (New Hire)

### Step 1 — Create the User Account

1. Navigate to **portal.azure.com** → Azure Active Directory → Users → New User
2. Select **"Create user"**
3. Fill in the following:
    - **Username:** [firstname.lastname@domain.com](mailto:firstname.lastname@domain.com)
    - **Display name:** First Last
    - **Password:** Auto-generate, check "Require password change at first login"
4. Click **Create**

### Step 2 — Assign Licenses

1. Open the new user profile → Click **"Licenses"** → **"Assignments"**
2. Assign the appropriate Microsoft 365 license (E3, E5, Business Premium, etc.)
3. Confirm the following services are enabled under the license:
    - Exchange Online (email)
    - Teams
    - SharePoint
    - OneDrive

### Step 3 — Assign Groups & Roles

1. Navigate to **Groups** → Add user to relevant department and security groups
2. If role requires admin access, assign the minimum necessary role under **"Assigned Roles"** — follow principle of least privilege
3. Document all group assignments in your ticketing system

### Step 4 — Configure MFA

1. Go to **aka.ms/mfasetup** or push enrollment via Conditional Access policy
2. Have user enroll Microsoft Authenticator as primary method
3. Set backup method: phone call or alternate email
4. Confirm MFA prompt fires successfully on first login

### Step 5 — Verify & Document

1. Test login with new credentials on a clean browser session
2. Confirm email is receiving, Teams is accessible, SharePoint loads
3. Close provisioning ticket with the following documented:
    - Account created ✅
    - License assigned ✅
    - Groups assigned ✅
    - MFA enrolled ✅
    - Verified by: [your name]

---

### Part 2 — User Offboarding (Termination or Resignation)

> ⚠️ **Security Note:** Offboarding must be initiated on or before the employee's last day. Never delay this workflow regardless of circumstances.
> 

### Step 1 — Disable the Account Immediately

1. Navigate to **Azure AD → Users** → search for the user
2. Click the user → **"Edit"** → set **Account Enabled** to **No**
3. This immediately blocks all sign-ins without deleting any data

### Step 2 — Revoke All Active Sessions

1. On the user profile page → click **"Revoke Sessions"**
2. This signs the user out of all devices and apps instantly
3. Confirm in the Audit Log that the revocation was applied

### Step 3 — Reset the Password

1. Reset the account password to a complex random string
2. Do not share this password with anyone — it is a security lockout measure

### Step 4 — Handle Email & Data

1. Set up an **email forwarding rule** to the user's manager (confirm with HR first)
2. Or convert the mailbox to a **Shared Mailbox** so the team retains access without consuming a license
3. Export OneDrive data if required — transfer ownership to manager

### Step 5 — Remove Licenses

1. Go to user profile → **Licenses** → remove all assigned licenses
2. This frees up the license for reassignment and stops billing on that seat

### Step 6 — Remove Group Memberships & App Access

1. Go to **Groups** → remove user from all security and Microsoft 365 groups
2. Go to **Enterprise Applications** → check for any third-party app assignments and revoke
3. Check **Conditional Access** policies for any user-specific rules referencing this account

### Step 7 — Document & Close

Document the following in your ticketing system before closing:

- Account disabled ✅
- Sessions revoked ✅
- Password reset ✅
- Email handled (forwarded / shared mailbox) ✅
- Licenses removed ✅
- Groups and app access revoked ✅
- Completed by: [your name] | Date: [date]

---

### Common Issues & Troubleshooting

**User still receiving emails after offboarding**
→ Check if forwarding rule was applied correctly. Also verify no mail flow rules in Exchange Admin Center are routing mail to the old address.

**License not releasing after removal**
→ Wait up to 24 hours. If still not released, check for conflicting group-based license assignments.

**Former employee still showing as active in a third-party app**
→ Azure AD revocation only covers Microsoft apps. Third-party apps (Salesforce, Slack, etc.) must be offboarded separately in each platform or via an SSO/SCIM integration.

**MFA prompt still appearing for a disabled account**
→ Confirm session revocation was applied. Check if the user has a separate B2B guest account that also needs to be disabled.

---

### Related Articles

- MFA Setup, Loss & Recovery
- Conditional Access Policy Troubleshooting
- Microsoft 365 License Management
- Password Reset & Account Unlock Procedures