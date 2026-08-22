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

<details>
<summary><b>1.1 — Forgotten Password (INC0010004)</b></summary>

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

</details>

<details>
<summary><b>1.2 — Forced Password Change at Logon Fails (INC0010005)</b></summary>

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

</details>

<details>
<summary><b>1.3 — Smart Card Logon Required Flag Set After Password Reset (INC0010006)</b></summary>

**Incident — INC0010006**

![Ticket](Phase-1-Password-Resets/1.3-SmartCard-Logon-Required/01-incident-inc0010006.png)

> "Someone from IT already reset my password today but I still cannot get into my computer. Now it says something about a smart card being required. I have never used a smart card here. I do not know what happened or what changed. My name is Elena Vance and I really need to get back into my system."

Caller: Elena Vance | Priority: 4 - Low | Group: Help Desk

---

**Reproduce**

Attempted to log in to MEL-CL-01 as ElenaV using the reset password. Login failed with a generic "The user name or password is incorrect" error. Windows did not explicitly present a smart card prompt at the login screen — the domain controller rejected password-based authentication entirely, which is the expected behavior when SmartcardLogonRequired is enabled on the account.

![Reproduce](Phase-1-Password-Resets/1.3-SmartCard-Logon-Required/03-reproduce-login-error.png)

---

**Isolate**

Opened Active Directory Users and Computers on MEL-DC-01. Located Elena Vance under MelvinLab_users. On the Account tab, "Smart card is required for interactive logon" was checked. This flag causes Windows to reject all password-based authentication at the domain controller level regardless of whether the password is correct — the login screen shows a generic failure rather than a smart card prompt, which makes this fault difficult to diagnose without checking the account directly in ADUC.

![Isolate](Phase-1-Password-Resets/1.3-SmartCard-Logon-Required/02-isolate-aduc-smartcard-checked.png)

---

**Resolve**

Unchecked "Smart card is required for interactive logon" on ElenaV's Account tab and applied the change. Then reset the password via right-click → Reset Password and enabled "User must change password at next logon" as a security precaution.

![Fix - Reset Password](Phase-1-Password-Resets/1.3-SmartCard-Logon-Required/04-fix-aduc-reset-password.png)

![Fix - Reset Confirmed](Phase-1-Password-Resets/1.3-SmartCard-Logon-Required/05-fix-password-reset-confirmed.png)

---

**Verify**

Elena was prompted to change her password at next logon — confirming authentication was restored and the forced change flow completed successfully.

![Verify - Must Change Password](Phase-1-Password-Resets/1.3-SmartCard-Logon-Required/06-verify-must-change-password.png)

![Verify - Elena Sets New Password](Phase-1-Password-Resets/1.3-SmartCard-Logon-Required/07-verify-elena-sets-new-password.png)

![Verify - Password Changed](Phase-1-Password-Resets/1.3-SmartCard-Logon-Required/08-verify-password-changed.png)

Logged in to MEL-CL-01 as Elena Vance with the new credentials. Desktop loaded successfully.

![Verify - Login Success](Phase-1-Password-Resets/1.3-SmartCard-Logon-Required/09-verify-login-success.png)

---

**Ticket Closed**

Resolution: Found "Smart card is required for interactive logon" flag enabled on ElenaV's AD account. This caused Windows to reject password-based authentication entirely, presenting as a generic login failure. Unchecked the SmartcardLogonRequired flag, reset the password, and enabled forced change at next logon. User authenticated and set a new password successfully.

![Resolved](Phase-1-Password-Resets/1.3-SmartCard-Logon-Required/10-ticket-resolved.png)

---

**KB Article — INC-KB-003: Smart Card Required Flag Blocks Login After Password Reset**

| | |
|---|---|
| **Symptom** | Password was reset by IT but user still cannot log in; error shows as generic "incorrect password" with no smart card prompt |
| **Check first** | ADUC → Account tab → "Smart card is required for interactive logon" |
| **Root cause** | SmartcardLogonRequired flag enabled on the account; domain controller rejects password-based authentication entirely |
| **Resolution** | ADUC → user properties → Account tab → uncheck SmartcardLogonRequired → reset password → confirm user can log in |
| **FCR** | Yes |
| **Time to resolve** | Under 10 minutes |

</details>

---

## Phase 2 — Account Lockouts

<details>
<summary><b>2.1 — Standard Account Lockout (INC0010025)</b></summary>

**Incident — INC0010025**

![Ticket](Phase-2-Account-Lockouts/2.1-Account-Lockout-Standard/01-incident-inc0010025.png)

> "My account is locked and I cannot log in this morning. I have not done anything different. My name is Marcus Thorne."

Caller: Marcus Thorne | Priority: 3 - Moderate | Group: Help Desk

---

**Reproduce**

Attempted to log in to MEL-CL-01 as MarcusT. Login failed immediately with "The referenced account is currently locked and may not be logged on to."

![Reproduce](Phase-2-Account-Lockouts/2.1-Account-Lockout-Standard/02-reproduce-login-error.png)

---

**Isolate**

Opened Active Directory Users and Computers on MEL-DC-01. Located Marcus Thorne under MelvinLab_users. On the Account tab, the message "Unlock account. This account is currently locked out on this Active Directory Domain Controller." confirmed the account was locked. The checkbox is unchecked by default — its presence and the accompanying message are the lockout indicators in ADUC.

![Isolate](Phase-2-Account-Lockouts/2.1-Account-Lockout-Standard/03-isolate-aduc-locked.png)

---

**Resolve**

Right-clicked Marcus Thorne in ADUC → Reset Password. Entered a temporary password, checked "Unlock account for user," and enabled "User must change password at next logon."

![Fix](Phase-2-Account-Lockouts/2.1-Account-Lockout-Standard/04-fix-aduc-reset-password.png)

![Password Changed Confirmation](Phase-2-Account-Lockouts/2.1-Account-Lockout-Standard/05-fix-password-changed.png)

---

**Verify**

Marcus was prompted to change his password at next logon — confirming the account was unlocked and authentication was restored.

![Verify - Must Change Password](Phase-2-Account-Lockouts/2.1-Account-Lockout-Standard/06-verify-must-change-password.png)

![Verify - Marcus Sets Password](Phase-2-Account-Lockouts/2.1-Account-Lockout-Standard/07-verify-marcus-sets-password.png)

Logged in to MEL-CL-01 as Marcus Thorne with the new credentials. Desktop loaded successfully.

![Verify - Login Success](Phase-2-Account-Lockouts/2.1-Account-Lockout-Standard/08-verify-login-success.png)

---

**Ticket Closed**

Resolution: Account confirmed locked in ADUC via Account tab lockout message. Unlocked account and reset password via ADUC. User was able to log in and set a new password successfully.

![Resolved](Phase-2-Account-Lockouts/2.1-Account-Lockout-Standard/09-ticket-resolved.png)

---

**KB Article — INC-KB-004: Standard Account Lockout**

| | |
|---|---|
| **Symptom** | User cannot log in; error states account is locked |
| **Check first** | ADUC → user properties → Account tab — look for "This account is currently locked out on this Active Directory Domain Controller." |
| **Root cause** | Account locked after exceeding failed logon threshold (policy: 5 attempts) |
| **Resolution** | ADUC → right-click user → Reset Password → check "Unlock account for user" → confirm user can log in |
| **FCR** | Yes |
| **Time to resolve** | Under 5 minutes |

</details>

<details>
<summary><b>2.2 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

<details>
<summary><b>2.3 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

---

## Phase 3 — DNS Resolution

<details>
<summary><b>3.1 — DNS Pointed Off Domain Controller (INC0010023)</b></summary>

**Incident — INC0010023**

![Ticket](Phase-3-DNS-Resolution/3.1-DNS-Pointed-Off-Domain-Controller/01-incident-inc0010023.png)

> "My computer stopped working this morning. I cannot get to any websites, I cannot access my network drives, I cannot reach anything on the network. It was all working fine yesterday and I have not changed any settings. My name is Liam O'Connor."

Caller: Liam O'Connor | Priority: 3 - Moderate | Group: Service Desk

---

**Reproduce**

Attempted to ping the domain controller by hostname from MEL-CL-01. The request failed immediately — "Ping request could not find host mel-dc-01.melvinlab.local. Please check the name and try again." The machine could not resolve any domain names, confirming the reported symptoms.

![Reproduce](Phase-3-DNS-Resolution/3.1-DNS-Pointed-Off-Domain-Controller/02-reproduce-ping-failure.png)

---

**Isolate**

Ran `ipconfig /all` on MEL-CL-01. The DNS Servers field under Ethernet adapter Ethernet 2 showed `10.10.10.99` — an IP address with no responding host on the network. The correct DNS server for this domain is the domain controller at `192.168.10.1`. With DNS pointed at a dead address, no hostname resolution is possible — network drives, domain resources, and internet access all fail at the DNS layer before any connection is attempted.

![Isolate](Phase-3-DNS-Resolution/3.1-DNS-Pointed-Off-Domain-Controller/03-isolate-ipconfig-bad-dns.png)

---

**Resolve**

Opened Network Connections (`ncpa.cpl`) on MEL-CL-01. Right-clicked Ethernet 2 → Properties → Internet Protocol Version 4 (TCP/IPv4) → Properties. Changed the Preferred DNS server from `10.10.10.99` to `192.168.10.1` (the domain controller) and clicked OK.

![Fix](Phase-3-DNS-Resolution/3.1-DNS-Pointed-Off-Domain-Controller/04-fix-dns-corrected.png)

---

**Verify**

Re-ran `ping mel-dc-01.melvinlab.local` from Command Prompt. The hostname resolved and replies came back from `192.168.10.1`, confirming DNS resolution was restored and the domain controller was reachable by name.

![Verify](Phase-3-DNS-Resolution/3.1-DNS-Pointed-Off-Domain-Controller/05-verify-ping-success.png)

---

**Ticket Closed**

Resolution: DNS server on the client adapter was misconfigured — pointing to `10.10.10.99`, a non-existent address on the network. Updated the preferred DNS server to `192.168.10.1` (MEL-DC-01). Hostname resolution restored immediately. All domain resources, network drives, and external connectivity returned to normal.

![Resolved](Phase-3-DNS-Resolution/3.1-DNS-Pointed-Off-Domain-Controller/06-ticket-resolved.png)

---

**KB Article — INC-KB-007: Client DNS Misconfigured — All Network Resources Unreachable**

| | |
|---|---|
| **Symptom** | User cannot reach any network resources, drives, or websites; ping by hostname fails immediately |
| **Check first** | `ipconfig /all` on client — look at DNS Servers field on the active adapter |
| **Root cause** | DNS server set to a non-existent IP; all hostname resolution fails silently at the DNS layer |
| **Resolution** | `ncpa.cpl` → adapter properties → TCP/IPv4 → set DNS to domain controller IP (`192.168.10.1`) |
| **FCR** | Yes |
| **Time to resolve** | Under 5 minutes |

</details>

<details>
<summary><b>3.2 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

<details>
<summary><b>3.3 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

---

## Phase 4 — Mapped Drives and File Share Access

<details>
<summary><b>4.1 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

<details>
<summary><b>4.2 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

<details>
<summary><b>4.3 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

---

## Phase 5 — DHCP and Network Connectivity

<details>
<summary><b>5.1 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

<details>
<summary><b>5.2 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

<details>
<summary><b>5.3 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

---

## Phase 6 — Printers

<details>
<summary><b>6.1 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

<details>
<summary><b>6.2 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

<details>
<summary><b>6.3 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

---

## Phase 7 — Group Policy Not Applying

<details>
<summary><b>7.1 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

<details>
<summary><b>7.2 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

<details>
<summary><b>7.3 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

---

## Phase 8 — Permissions and Access Denied

<details>
<summary><b>8.1 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

<details>
<summary><b>8.2 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

<details>
<summary><b>8.3 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

---

## Phase 9 — Software Deployment Failures

<details>
<summary><b>9.1 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

<details>
<summary><b>9.2 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

<details>
<summary><b>9.3 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

---

## Phase 10 — Data Recovery

<details>
<summary><b>10.1 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

<details>
<summary><b>10.2 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>

<details>
<summary><b>10.3 — [TBD]</b></summary>

*This ticket has not yet been documented.*

</details>
