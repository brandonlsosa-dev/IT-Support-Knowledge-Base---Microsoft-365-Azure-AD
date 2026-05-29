# Microsoft 365 License Management

Category: Microsoft 365
Difficulty: Tier 2
Last Updated: May 2026
Environment: Microsoft 365 Admin Center | Azure AD / Entra ID | Remote Support

---

### Overview

Microsoft 365 license management is a core IT responsibility that directly impacts both security and company spend. Unmanaged licenses mean paying for inactive accounts, blocked users who need access, and compliance risks from improper role assignments.

This article covers assigning, removing, and auditing licenses across a Microsoft 365 environment — including group-based licensing for scale.

---

### Prerequisites

Before managing licenses confirm you have:

- License Administrator, User Administrator, or Global Administrator role
- Access to Microsoft 365 Admin Center (**admin.microsoft.com**)
- Access to Azure AD / Entra ID portal
- Current license inventory from your billing admin or IT manager

---

### Part 1 — Understanding License Types

The most common Microsoft 365 license tiers you'll encounter in IT support:

| License | Key Apps Included | Typical User |
| --- | --- | --- |
| Microsoft 365 Business Basic | Exchange, Teams, SharePoint (web only) | Frontline / light users |
| Microsoft 365 Business Standard | Full Office apps + Teams + Exchange | General staff |
| Microsoft 365 Business Premium | Standard + Intune + Azure AD P1 + Defender | IT-managed devices |
| Microsoft 365 E3 | Full Office + compliance tools + Azure AD P1 | Enterprise standard |
| Microsoft 365 E5 | E3 + advanced security + Power BI + voice | Enterprise premium / security teams |

> **Key rule:** Always assign the minimum license that meets the user's needs. Over-licensing is one of the most common and costly mistakes in Microsoft 365 environments.
> 

---

### Part 2 — Assigning a License to a Single User

### Via Microsoft 365 Admin Center

1. Navigate to **admin.microsoft.com** → Users → Active Users
2. Search for and open the user
3. Click the **"Licenses and apps"** tab
4. Check the box next to the appropriate license
5. Expand the license to confirm which apps are enabled — disable any the user doesn't need
6. Click **"Save changes"**
7. Allow up to 15 minutes for all services to provision

### Via Azure AD Portal

1. Navigate to **portal.azure.com** → Azure Active Directory → Users
2. Open the user → click **"Licenses"** → **"Assignments"**
3. Click **"+"** → select the license → click **"Save"**
4. Verify assigned apps under the license detail view

### What to Document in Your Ticket

- License type assigned
- Apps enabled/disabled
- Business justification if assigning above standard tier
- Assigned by and date

---

### Part 3 — Removing a License

### When to Remove

- User is offboarded
- User role changed and no longer needs current license tier
- License reclamation audit identified inactive accounts

### Steps

1. Navigate to **admin.microsoft.com** → Users → Active Users
2. Open the user → **"Licenses and apps"** tab
3. Uncheck the license → click **"Save changes"**
4. If offboarding: confirm mailbox handling before removing Exchange license — removing it starts a 30-day countdown before the mailbox is permanently deleted

> ⚠️ **Critical:** Removing an Exchange Online license does not immediately delete the mailbox — but it starts a 30-day grace period. Convert to a shared mailbox first if the data needs to be retained beyond that window.
> 

### Reclaiming Licenses at Scale

1. Navigate to **admin.microsoft.com** → Billing → Licenses
2. Review the **"Unassigned licenses"** count per product
3. Cross-reference with Azure AD Sign-In Logs to find accounts with no sign-in activity in 90+ days
4. Disable those accounts → remove licenses → document in your asset tracker

---

### Part 4 — Group-Based Licensing

For environments with 20+ users, managing licenses individually is inefficient. Group-based licensing assigns licenses automatically based on group membership.

### Setting Up Group-Based Licensing

1. Navigate to **portal.azure.com** → Azure Active Directory → Groups
2. Open or create the target group (e.g. "M365-BusinessPremium-Users")
3. Click **"Licenses"** → **"Assignments"**
4. Select the license to assign → configure which apps to enable
5. Click **"Save"**
6. Any user added to this group automatically receives the license
7. Any user removed from this group automatically loses the license

### Benefits

- Eliminates manual license assignment errors
- Automatically reclaims licenses when users are removed from groups
- Simplifies onboarding — adding a user to the right group handles licensing automatically
- Provides a clean audit trail via group membership history

### Troubleshooting Group-Based Licensing Errors

1. Navigate to **Azure AD → Groups → [Group] → Licenses**
2. Check for any users shown with licensing errors
3. Common causes:
    - Not enough available licenses in the tenant — purchase more or reclaim unused
    - User already has a conflicting license assigned directly — remove the direct assignment
    - License includes a service plan that conflicts with another assigned license

---

### Part 5 — License Auditing

Run a license audit at least quarterly. This keeps costs controlled and ensures compliance.

### Quick Audit via Admin Center

1. Navigate to **admin.microsoft.com** → Billing → Licenses
2. Review each product:
    - Total purchased vs. total assigned
    - Any products at 100% capacity — flag for potential purchase
    - Any products with large numbers of unassigned licenses — flag for reclamation

### Inactive User Audit

1. Navigate to **Azure AD → Users** → filter by **"Sign-in activity"**
2. Export users with no sign-in in the last 90 days
3. Cross-reference with HR to confirm employment status
4. Disable accounts for departed employees → remove licenses
5. Document findings and actions taken in your ticketing system

### License Report via Microsoft 365 Admin Center

1. Navigate to **admin.microsoft.com** → Reports → Usage
2. Open **"Microsoft 365 active users"** report
3. Review per-app activity — identify users with licenses for apps they never use
4. Downgrade those users to a lower license tier where appropriate

---

### Part 6 — Common Issues & Troubleshooting

**User assigned a license but apps not showing up**
→ Wait 15–30 minutes for provisioning. If still not showing after an hour, remove and reassign the license. Check if specific app services were accidentally disabled under the license assignment.

**User can't access Teams after license assigned**
→ Confirm Teams service plan is enabled under the license. Also check if the user is in a Teams-disabled policy under Teams Admin Center → Users.

**License assignment failing with an error**
→ Check available license count — you may be out of seats. Navigate to Billing → Licenses to confirm inventory. Also check for conflicting direct vs. group-based assignments.

**Shared mailbox consuming a license**
→ Shared mailboxes under 50GB do not require a license. If a license is assigned to a shared mailbox, remove it — the mailbox will continue functioning as long as it stays under the size limit.

**User showing as unlicensed but can still access apps**
→ Grace period — Microsoft provides a short window after license removal where access continues. Confirm the license was fully removed and allow 24 hours for access to fully revoke.

---

### Related Articles

- Azure AD User Provisioning & Offboarding
- Conditional Access Policy Troubleshooting
- Outlook & Teams Common Issues