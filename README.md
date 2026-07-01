<div align="center">
<img width="300" height="150" alt="images" src="https://github.com/user-attachments/assets/fcd0d725-24e8-4d33-99e2-65587b54047d" />
</div>


# Active Directory & Entra ID Hybrid IAM Implementation

A hands-on Identity and Access Management lab built to simulate real-world enterprise hybrid identity workflows using Windows Server 2025, Active Directory, and Microsoft Entra ID. This project covers the full IAM lifecycle — from environment deployment and user provisioning to privileged access management, conditional access enforcement, and security monitoring.

---

## Table of Contents

- [Overview](#overview)
- [Technologies Used](#technologies-used)
- [Architecture](#architecture)
- [Project Phases](#project-phases)
  - [Phase 1 — Environment Setup](#phase-1--environment-setup)
  - [Phase 2 — OU Structure & User Provisioning](#phase-2--ou-structure--user-provisioning)
  - [Phase 3 — Role Based Access Control](#phase-3--role-based-access-control)
  - [Phase 4 — Conditional Access Policies](#phase-4--conditional-access-policies)
  - [Phase 5 — Privileged Identity Management](#phase-5--privileged-identity-management)
  - [Phase 6 — Password & Authentication Policies](#phase-6--password--authentication-policies)
  - [Phase 7 — Monitoring & Auditing](#phase-7--monitoring--auditing)
- [Key Skills Demonstrated](#key-skills-demonstrated)

---

## Overview

This lab replicates the hybrid identity infrastructure found in enterprise environments where on-premises Active Directory is synchronized to Microsoft Entra ID (formerly Azure AD). The project demonstrates end-to-end IAM operations including user lifecycle management, RBAC implementation, privileged access controls, and security monitoring — all automated where possible using PowerShell and the Microsoft Graph API.

---

## Technologies Used

| Technology | Purpose |
|---|---|
| Windows Server 2025 | Domain Controller / AD DS |
| Active Directory Domain Services | On-premises identity store |
| Microsoft Entra ID | Cloud identity provider |
| Microsoft Entra Connect Sync | Hybrid identity synchronization |
| PowerShell | Automation and scripting |
| Microsoft Graph API | Entra ID management and reporting |
| Azure Log Analytics | Security monitoring |
| KQL (Kusto Query Language) | Log querying and threat detection |
| Privileged Identity Management (PIM) | Just-in-time privileged access |

---

## Architecture

```
┌─────────────────────────────┐         ┌──────────────────────────────┐
│   On-Premises Environment   │         │     Microsoft Entra ID       │
│                             │         │                              │
│  Windows Server 2025 DC     │◄───────►│  Cloud Identity & Access     │
│  AD DS (examlabpractice.com)│  Entra  │  Conditional Access          │
│  500+ User Accounts         │ Connect │  PIM                         │
│  Department OUs             │  Sync   │  RBAC                        │
│  Security Groups            │         │  Log Analytics               │
│  Fine-Grained Pwd Policies  │         │  SSPR                        │
└─────────────────────────────┘         └──────────────────────────────┘
```

---

## Project Phases

### Phase 1 — Environment Setup

- Deployed Windows Server 2025 and promoted to Domain Controller
- Configured Active Directory Domain Services with domain `examlabpractice.com`
- Bulk-provisioned 500 randomized test user accounts using a custom PowerShell script (`New-BulkADUsers.ps1`)
- Installed and configured Microsoft Entra Connect Sync for hybrid identity
- Verified bidirectional sync — confirmed all users appear in Entra ID portal
 <img width="850" height="658" alt="Screenshot 2026-06-28 at 4 42 31 PM" src="https://github.com/user-attachments/assets/e493f4d0-09e7-4bb2-acb3-468f207cd0d1" />
<img width="1475" height="756" alt="Screenshot 2026-06-28 at 4 44 09 PM" src="https://github.com/user-attachments/assets/7b34bb4f-e7ac-43ea-9b14-f650c755ac62" />





---

### Phase 2 — OU Structure & User Provisioning

- Created department-based OU structure (IT, HR, Finance, Sales, Disabled) via PowerShell
- Created department security groups via PowerShell
- Provisioned users into correct OUs by department with proper AD attributes
- Automated onboarding workflow — single script creates user, assigns group
- Automated offboarding workflow — disables account, strips group memberships, moves to Disabled OU
- Verified all changes reflected in Entra ID after each sync cycle
<img width="1268" height="722" alt="Screenshot 2026-06-28 at 5 49 04 PM" src="https://github.com/user-attachments/assets/1322a5d1-2c5b-4da8-bb82-e93d1742d8b7" />




### Phase 3 — Role Based Access Control

- Created Help Desk security group and assigned Password Administrator role in Entra ID
- Created IT Admins security group and assigned User Administrator role in Entra ID
- Used PowerShell and Microsoft Graph API to query all role assignments and export to CSV
- Documented least privilege justification for each role assignment
- Tested role boundaries — verified Help Desk can reset passwords but cannot create users

  ## Role Assignment — Least Privilege Justification

| Role | Assigned To | Justification |
|---|---|---|
| Password Administrator | HelpDesk | Allows Help Desk to reset user passwords without granting broader user management rights |
| User Administrator | ITAdmins | Allows IT Admins to create and manage users without granting Global Administrator privileges |
| Directory Readers | Service Principal | Read-only access required for Entra Connect sync operations |
| Global Administrator | Admin accounts | Break-glass accounts only — not used for day-to-day operations |
<img width="1848" height="749" alt="Screenshot 2026-06-29 at 12 15 43 PM" src="https://github.com/user-attachments/assets/1dd4fe72-99e6-4c9c-bb50-c159a8b3319f" />




### Phase 4 — Conditional Access Policies

- Created policy requiring MFA for all users on all cloud applications
- Created policy blocking sign-ins from outside the United States using named locations
- Configured named location for the lab environment
- Tested policies in Report-Only mode before enforcement
- Documented business justification and threat coverage for each policy

  ## Conditional Access Policies

| Policy | Scope | Grant Control | Mode | Business Justification |
|---|---|---|---|---|
| Require MFA - All Users All Apps | All users, All cloud apps | Require MFA | Report-only | Mitigates credential theft and phishing attacks by requiring a second factor for all cloud access regardless of location |
| Block Sign-ins - Outside United States | All users, All cloud apps | Block access | Report-only | Reduces attack surface by preventing authentication attempts from outside the organization's operating geography |

### Threat Coverage
- **Credential stuffing** — MFA policy ensures stolen passwords alone cannot grant access
- **Phishing** — MFA requirement adds friction even when credentials are compromised
- **Foreign threat actors** — Location-based block prevents sign-ins from outside the US
- **Lateral movement** — Combined policies limit blast radius if an account is compromised

### Report-Only Mode Justification
Both policies were deployed in Report-only mode first to evaluate impact on existing users and service accounts before enforcement, following least-disruption change management practices.
<img width="890" height="863" alt="Screenshot 2026-06-29 at 5 12 49 PM" src="https://github.com/user-attachments/assets/f059675d-d81b-4d1c-8608-8f4d736227f6" />


---

### Phase 5 — Privileged Identity Management

- Enabled PIM in Entra ID and converted Global Administrator to an eligible role
- Configured activation requirements — MFA and written justification required
- Set maximum activation duration to 1 hour
- Configured approval workflow requiring a second admin to approve activation
- Tested the full just-in-time flow — requested access, approved, verified, expired
- Documented how PIM reduces standing admin access risk and lateral movement exposure

<img width="1970" height="1118" alt="Screenshot 2026-06-30 at 2 17 47 PM" src="https://github.com/user-attachments/assets/7560a737-634b-483b-9a79-58d7c236101c" />

## Privileged Identity Management (PIM) Configuration

### How PIM Reduces Standing Admin Access Risk

| Risk | Without PIM | With PIM |
|---|---|---|
| Standing access | Global Admin active 24/7 | Access only active when needed |
| Credential compromise | Attacker inherits full admin rights immediately | Attacker cannot activate without MFA + justification + approval |
| Lateral movement | Compromised admin can pivot across entire tenant | 1-hour window limits blast radius |
| Insider threat | No audit trail for admin actions | Every activation logged with justification and approver |
| Over-provisioning | Admins accumulate roles over time | Eligible assignments require explicit activation each time |

### Activation Requirements Configured
- Azure MFA required on activation
- Written justification required
- Second admin approval required
- Maximum activation duration: 1 hour


---

### Phase 6 — Password & Authentication Policies

- Created Fine-Grained Password Policy for admin accounts via PowerShell:
  - 16 character minimum length
  - Complexity required
  - 60 day expiration
  - 10 password history
- Applied policy to IT Admins security group via PowerShell
- Enabled Entra ID Password Protection to block weak and organization-banned passwords
- Configured Self-Service Password Reset (SSPR) for all users
- Tested and documented the full SSPR registration and reset flow

<img width="1102" height="735" alt="Screenshot 2026-06-30 at 2 34 47 PM" src="https://github.com/user-attachments/assets/5b997f8f-b016-40f7-a0f5-841fd92a6fdc" />

## Password Management Configuration

### Fine-Grained Password Policy — AdminPasswordPolicy
Applied to: `ITAdmins` security group via PowerShell

| Setting | Value |
|---|---|
| Minimum password length | 16 characters |
| Complexity required | Yes |
| Maximum password age | 60 days |
| Password history | 10 passwords |
| Precedence | 10 |

### Entra ID Password Protection
| Setting | Value |
|---|---|
| Mode | Audit |
| Custom banned passwords | Enabled |
| On-premises AD protection | Enabled |

### Self-Service Password Reset (SSPR)
| Setting | Value |
|---|---|
| Enabled for | All users |
| Methods required to reset | 2 |
| Authentication methods | Mobile app notification, Email, Mobile phone |
| Registration required at sign-in | Yes |
| Re-confirmation interval | 180 days |

### SSPR Flow Tested
- User navigated to https://aka.ms/sspr
- Registered authentication methods on first sign-in
- Successfully completed identity verification
- Password reset without admin intervention




### Phase 7 — Monitoring & Auditing

- Created Log Analytics Workspace in Azure
- Connected Entra ID diagnostic logs (sign-in logs, audit logs) to Log Analytics
- Wrote KQL queries for threat detection:
  - Failed sign-in attempts in the last 24 hours
  - Users added to privileged roles
  - Sign-ins from outside the United States
- Used PowerShell and Microsoft Graph to generate privileged role membership report
- Used PowerShell to pull and export Entra ID audit log events to CSV
- Wrote a mock incident report based on simulated findings

  <img width="960" height="709" alt="Screenshot 2026-07-01 at 12 40 50 PM" src="https://github.com/user-attachments/assets/424dd4e7-249d-43e8-a96d-e55e1240d950" />

## Mock Incident Report

**Incident ID:** INC-2026-001
**Date:** June 30, 2026
**Severity:** Medium
**Status:** Closed — Simulated/Lab Exercise

### Summary
During routine audit log review, multiple failed password reset attempts were detected 
for a user account, followed by a successful self-service password reset. Additionally, 
a privileged role activation request was submitted and approved within a short timeframe.

### Timeline of Events
| Time | Event | User | Result |
|---|---|---|---|
| 9:09 AM | Change password (self-service) | pim-test | success |
| 9:10 AM | User started security info registration | pim-test | success |
| 9:14 AM | Add member to role requested (PIM) | pim-test | success |
| 9:17 AM | Request approved | iam-admin | success |
| 9:18 AM | Add member to role completed (PIM) | pim-test | success |
| 9:43 AM | Reset password (self-service) | helpdesktester | failure |

### Findings
- PIM activation flow completed successfully with MFA and approval — expected behavior
- SSPR registration and reset flow functioning as configured
- Failed password reset attempt logged and auditable

### Containment
No containment required — all activity consistent with lab testing procedures.

### Recommendations
- Monitor for repeated failed SSPR attempts as potential account enumeration
- Ensure PIM activation justifications are reviewed regularly
- Set alerts in Log Analytics for failed sign-ins exceeding threshold




## Key Skills Demonstrated

- Hybrid identity architecture and Entra Connect Sync configuration
- Active Directory administration — OUs, groups, and fine-grained password policies
- PowerShell automation for user lifecycle management (provisioning, onboarding, offboarding)
- Microsoft Graph API integration for identity reporting and auditing
- Role Based Access Control (RBAC) with least privilege principles
- Conditional Access policy design and enforcement
- Privileged Identity Management (PIM) with just-in-time access and approval workflows
- Security monitoring using KQL queries in Azure Log Analytics
- Incident documentation and audit log analysis
