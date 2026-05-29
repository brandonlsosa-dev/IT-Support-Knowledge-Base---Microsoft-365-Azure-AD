# MFA Setup, Loss & Recovery

Category: Identity & Access
Difficulty: Tier 1-2
Last Updated: May 2026
Environment: Microsoft 365 / Entra ID | Microsoft Authenticator | Remote Support

---

### Overview

Multi-factor authentication is the single most effective control against unauthorized account access. As an IT support specialist, MFA-related tickets are among the highest volume you will handle — covering initial enrollment, device changes, lost access, and policy enforcement.

This article covers the full MFA lifecycle: setup, common issues, and recovery procedures with proper identity verification steps.

---

### Prerequisites

Before performing any MFA reset or recovery, confirm you have:

- User Administrator or Authentication Administrator role in Azure AD
- A verified method of confirming the user's identity before making any changes
- Access to the Microsoft 365 Admin Center or Azure AD / Entra ID portal

> ⚠️ **Security Note:** Never reset MFA without first verifying the user's identity. MFA resets are a high-value social engineering target. Always confirm via employee ID, manager verification, or HR record before proceeding.
> 

---

### Part 1 — New User MFA Enrollment

### Method 1 — Self-Service via aka.ms/mfasetup

1. Direct the user to **aka.ms/mfasetup** in a browser
2. Sign in with their Microsoft 365 credentials
3. Click **"Add sign-in method"** → select **"Authenticator app"**
4. On their mobile device: download **Microsoft Authenticator** from the App Store or Google Play
5. In the app: tap **"+"** → **"Work or school account"** → **"Scan QR code"**
6. Scan the QR code displayed on screen
7. Complete the verification test prompt — enter the number shown on screen into the app
8. Set a backup method: phone call or alternate email
9. Confirm with the user that the next login prompts for MFA successfully

### Method 2 — Enforced via Conditional Access Policy

1. Navigate to **Azure AD → Security → Conditional Access**
2. Open the relevant policy requiring MFA
3. Confirm the new user falls within the policy scope (All Users or specific group)
4. User will be prompted to enroll on next login automatically
5. Monitor the **Sign-In Logs** to confirm enrollment completed

---

### Part 2 — MFA Loss (New Device or Lost Phone)

This is the most common MFA ticket. Handle in this order:

### Step 1 — Verify User Identity

Confirm at least **two** of the following before proceeding:

- Employee ID number
- Manager name and confirmation from manager directly
- HR record match (date of birth, start date, or similar)
- Government-issued ID if in-person

Document your verification method in the ticket.

### Step 2 — Temporarily Bypass MFA

1. Navigate to **Azure AD → Users** → search for the user
2. Click the user → **"Authentication Methods"**
3. Click **"Require re-register MFA"** — this forces re-enrollment on next login without fully disabling MFA
4. Alternatively: go to **aka.ms/mfasetup** as admin and delete the existing authenticator entry

### Step 3 — Have User Re-Enroll

1. Direct user to **aka.ms/mfasetup**
2. Walk them through adding Microsoft Authenticator on their new device (see Part 1)
3. Verify the new enrollment fires a successful prompt before closing the ticket

### Step 4 — Document & Close

- Identity verification method used ✅
- Old MFA method removed ✅
- New MFA method enrolled ✅
- Tested successfully ✅
- Completed by: [your name] | Date: [date]

---

### Part 3 — MFA Recovery (Locked Out Completely)

When a user has no access to any MFA method and cannot self-recover:

### Step 1 — Verify Identity (Stricter Standard)

Because this is a full lockout, require **manager confirmation AND one additional factor** before proceeding. Document everything.

### Step 2 — Disable MFA Temporarily

1. Navigate to **Azure AD → Users** → open the user profile
2. Go to **"Authentication Methods"** → delete all registered methods
3. This removes MFA enforcement for that user temporarily

### Step 3 — Create a Temporary Access Pass (Recommended)

1. Navigate to **Azure AD → Users** → open the user
2. Click **"Authentication Methods"** → **"Add authentication method"**
3. Select **"Temporary Access Pass"**
4. Set duration: 1 hour is sufficient for re-enrollment
5. Share the TAP with the user via a secure channel — never email or Teams if the account is compromised
6. User logs in with TAP → immediately re-enrolls MFA

### Step 4 — Re-Enable MFA Enforcement

1. Confirm new MFA method is enrolled and tested
2. Confirm Conditional Access policy is re-applying to the user
3. Close ticket with full documentation

---

### Part 4 — Common Issues & Troubleshooting

**"I'm not receiving the MFA prompt"**
→ Check if the user is excluded from the Conditional Access policy. Check if their account has MFA disabled at the per-user level overriding the policy.

**Authenticator app showing wrong code**
→ Time sync issue. In the Authenticator app: Settings → Time Correction for Codes → Sync Now. Codes are time-based (TOTP) and fail if the phone clock is off by more than 30 seconds.

**User getting MFA prompt on every login despite "Remember this device" being checked**
→ Check Conditional Access policy — "Sign-in frequency" setting may be overriding the remember device setting. Also check if the user is clearing browser cookies between sessions.

**MFA prompt firing but approval not going through**
→ Check internet connectivity on the user's mobile device. Microsoft Authenticator requires an active connection to approve push notifications. Fallback: use the 6-digit code instead of the push notification.

**New phone, Authenticator transferred but codes not working**
→ The backup/transfer feature in Authenticator does not transfer TOTP codes for work accounts due to security policy. User must re-enroll from scratch — follow Part 2.

**User receiving MFA prompts they didn't initiate**
→ Potential account compromise. Do not reset MFA — escalate immediately. Disable the account, revoke sessions, and notify your security team. Review Sign-In Logs for suspicious activity.

---

### Security Best Practices

- Always use **Microsoft Authenticator push notifications** as the primary method over SMS — SMS is vulnerable to SIM swapping
- Enforce MFA for all users via **Conditional Access** rather than per-user MFA settings — it's more flexible and auditable
- Require MFA re-enrollment after any device change
- Review MFA registration reports monthly: **Azure AD → Security → Authentication Methods → Registration Details**
- Never bypass MFA for convenience — every exception is a security risk

---

### Related Articles

- Azure AD User Provisioning & Offboarding
- Conditional Access Policy Troubleshooting
- Password Reset & Account Unlock Procedures