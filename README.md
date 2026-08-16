![Project Status](https://img.shields.io/badge/Project-In%20Progress-yellow)
![Last Update](https://img.shields.io/badge/Last%20Update-August%202026-blue)
![License](https://img.shields.io/badge/License-MIT-green)
![Tech Stack](https://img.shields.io/badge/Tech-ServiceNow%20%7C%20Windows%20Server%202022%20%7C%20Active%20Directory-lightgrey)

# ServiceNow ITSM Operations Lab

This lab documents real incident resolution work performed against a live Windows domain environment. Faults are injected into `melvinlab.local`, submitted as incidents in a ServiceNow Personal Developer Instance, and worked end to end — reproduce, isolate, resolve, verify, document. Every ticket in this repository was actually worked. None were authored and closed without the diagnostic steps being performed.

The environment is `melvinlab.local` running on Proxmox VE 9.1.0 — MEL-DC-01 (Windows Server 2022 Domain Controller) and MEL-CL-01 (Windows 11 client), with 20 users across departmental OUs. The ServiceNow instance is hosted by ServiceNow (dev374741.service-now.com, Australia release) and connects to that environment as the ticketing layer.

The organizing principle is category over quantity. Real service desk work is not thirty unrelated problems — it is a handful of recurring categories that present differently each time. Each phase covers one category through three variations, escalating in difficulty. The third variation in every phase is deliberately one where the obvious fix does not resolve the issue, because recognizing that is the actual skill.

---

## Table of Contents

- [Phase 1: Password Resets](#phase-1-password-resets)
- [Phase 2: Account Lockouts](#phase-2-account-lockouts)
- [Phase 3: DNS Resolution](#phase-3-dns-resolution)
- [Phase 4: Mapped Drives and File Share Access](#phase-4-mapped-drives-and-file-share-access)
- [Phase 5: DHCP and Network Connectivity](#phase-5-dhcp-and-network-connectivity)
- [Phase 6: Printers](#phase-6-printers)
- [Phase 7: Group Policy Not Applying](#phase-7-group-policy-not-applying)
- [Phase 8: Permissions and Access Denied](#phase-8-permissions-and-access-denied)
- [Phase 9: Software Deployment Failures](#phase-9-software-deployment-failures)
- [Phase 10: Data Recovery](#phase-10-data-recovery)

---

## Phase 1 — Password Resets

The most common ticket on any service desk. A reset is three clicks — knowing *why* the account cannot authenticate is the job.

---

### 1.1 — Forgotten Password

**Incident — INC0010004**

![Ticket](Phase-1-Password-Resets/1.1-Forgotten-Password/01-incident-inc0010004.png)

> "Hi, I've been trying to log into my computer all morning and it keeps saying my password is wrong. I haven't changed it or anything, it just stopped working. I need to get in to do my work. Can someone please help me reset it? My name is Jordan Rivera."

Caller: Jordan Rivera | Priority: 3 - Moderate | Group: Help Desk

---

**Reproduce**

Attempted to log in to MEL-CL-01 as JordanR. Login failed immediately with "The user name or password is incorrect."

![Reproduce](Phase-1-Password-Resets/1.1-Forgotten-Password/02-reproduce-login-error.png)

---

**Isolate**

Opened Active Directory Users and Computers on MEL-DC-01. Located Jordan Rivera under MelvinLab_users. Checked the Account tab — account was active, not locked, no flags set. The account looked completely normal from the outside. The issue was not the account state; the password itself had been changed without the user's knowledge.

![Isolate](Phase-1-Password-Resets/1.1-Forgotten-Password/03-isolate-aduc-account.png)

---

**Resolve**

Right-clicked Jordan Rivera in ADUC → Reset Password. Set a new temporary password and confirmed the reset.

![Fix](Phase-1-Password-Resets/1.1-Forgotten-Password/04-fix-password-reset.png)

Jordan was prompted to set a new password at first login — confirming the reset took effect.

![Password Change Prompt](Phase-1-Password-Resets/1.1-Forgotten-Password/05-must-change-password-prompt.png)

![Jordan Sets New Password](Phase-1-Password-Resets/1.1-Forgotten-Password/06-jordan-sets-new-password.png)

---

**Verify**

Logged in to MEL-CL-01 as Jordan Rivera with the new credentials. Desktop loaded successfully.

![Verify](Phase-1-Password-Resets/1.1-Forgotten-Password/07-verify-desktop.png)

---

**Ticket Closed**

Resolution: Verified account was active and unlocked in ADUC but password was invalid. Reset password to temporary value and confirmed user was able to log in successfully.

![Resolved](Phase-1-Password-Resets/1.1-Forgotten-Password/08-ticket-resolved.png)

---

**KB Article — INC-KB-001: Password Reset, Standard**

| | |
|---|---|
| **Symptom** | User cannot log in; reports password stopped working without any changes on their end |
| **Check first** | ADUC → Account tab — is the account locked, disabled, or flagged? |
| **Root cause** | Password was changed externally; user was not notified |
| **Resolution** | ADUC → right-click user → Reset Password → set temporary password → confirm user can log in |
| **FCR** | Yes |
| **Time to resolve** | Under 5 minutes |

---

### 1.2 — Forced Password Change at Logon Fails

**Incident — INC0010005**

![Ticket](Phase-1-Password-Resets/1.2-Forced-Password-Change-At-Logon/01-incident-inc0010005.png)

> "I called in last week about my password and was given a temporary one. Now every time I try to log in it says I have to change my password, I try to change it, and then it just gives me an error and kicks me back to the login screen. I cannot get in at all. My name is Kevin Park."

Caller: Kevin Park | Priority: 3 - Moderate | Group: Help Desk

---

**Reproduce**

Attempted to log in to MEL-CL-01 as KevinP using the temporary password. Login failed with "The password is incorrect."

![Reproduce](Phase-1-Password-Resets/1.2-Forced-Password-Change-At-Logon/04-reproduce-login-error.png)

When attempting with the correct temp password, the system prompted a forced password change but then returned an error and kicked back to the login screen.

![Forced Change Prompt](Phase-1-Password-Resets/1.2-Forced-Password-Change-At-Logon/08-verify-user-prompted-to-change.png)

---

**Isolate**

Opened Active Directory Users and Computers on MEL-DC-01. Located Kevin Park under MelvinLab_users. On the Account tab, "User must change password at next logon" was checked. The flag forces a password change at login but the process was failing, leaving Kevin unable to authenticate at all.

![Isolate](Phase-1-Password-Resets/1.2-Forced-Password-Change-At-Logon/02-isolate-aduc-kevinp-must-change-flag.png)

---

**Resolve**

Right-clicked Kevin Park in ADUC → Reset Password. Set a new temporary password. Confirmed the reset was applied in AD.

![Fix](Phase-1-Password-Resets/1.2-Forced-Password-Change-At-Logon/05-fix-aduc-reset-password.png)

![Password Reset Confirmed](Phase-1-Password-Resets/1.2-Forced-Password-Change-At-Logon/06-fix-password-change-confirmed.png)

Verified account settings after reset.

![Verify Settings](Phase-1-Password-Resets/1.2-Forced-Password-Change-At-Logon/07-fix-aduc-verify-settings.png)

Kevin was prompted to set a new password at login — confirming the reset took effect and the forced change flow completed successfully.

![Kevin Changes Password](Phase-1-Password-Resets/1.2-Forced-Password-Change-At-Logon/09-verify-password-changed-success.png)

---

**Verify**

Logged in to MEL-CL-01 as Kevin Park with the new credentials. Desktop loaded successfully.

![Verify](Phase-1-Password-Resets/1.2-Forced-Password-Change-At-Logon/10-verify-login-success.png)

---

**Ticket Closed**

Resolution: Found "User must change password at next logon" flag set on KevinP's AD account. Reset password via ADUC and confirmed user was able to complete the password change and log in successfully.

![Resolved](Phase-1-Password-Resets/1.2-Forced-Password-Change-At-Logon/11-ticket-resolved.png)

---

**KB Article — INC-KB-002: Forced Password Change at Logon Fails**

| | |
|---|---|
| **Symptom** | User was given a temp password but gets kicked back to the login screen when trying to change it |
| **Check first** | ADUC → Account tab — is "User must change password at next logon" checked? |
| **Root cause** | pwdLastSet = 0; forced change at logon flag set, password change process failing at login screen |
| **Resolution** | ADUC → right-click user → Reset Password → set new temp password → confirm user completes change at login |
| **FCR** | Yes |
| **Time to resolve** | Under 10 minutes |
