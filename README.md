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

## Phase 1: Password Resets

*Coming soon*

---

## Phase 2: Account Lockouts

*Coming soon*

---

## Phase 3: DNS Resolution

*Coming soon*

---

## Phase 4: Mapped Drives and File Share Access

*Coming soon*

---

## Phase 5: DHCP and Network Connectivity

*Blocked — requires AD Phase 8 (DHCP role)*

---

## Phase 6: Printers

*Blocked — requires print services build*

---

## Phase 7: Group Policy Not Applying

*Blocked — requires AD Phase 7 (GPO software deployment)*

---

## Phase 8: Permissions and Access Denied

*Coming soon*

---

## Phase 9: Software Deployment Failures

*Blocked — requires AD Phase 7 (GPO software deployment)*

---

## Phase 10: Data Recovery

*Coming soon*
