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

---

### Phase 5 — Privileged Identity Management

- Enabled PIM in Entra ID and converted Global Administrator to an eligible role
- Configured activation requirements — MFA and written justification required
- Set maximum activation duration to 1 hour
- Configured approval workflow requiring a second admin to approve activation
- Tested the full just-in-time flow — requested access, approved, verified, expired
- Documented how PIM reduces standing admin access risk and lateral movement exposure

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

**Scripts:** `New-AdminPasswordPolicy.ps1`

**Key commands:**
```powershell
# Create Fine-Grained Password Policy
New-ADFineGrainedPasswordPolicy -Name "AdminPolicy" `
    -Precedence 10 `
    -MinPasswordLength 16 `
    -ComplexityEnabled $true `
    -MaxPasswordAge "60.00:00:00" `
    -PasswordHistoryCount 10

# Apply to IT Admins group
Add-ADFineGrainedPasswordPolicySubject -Identity "AdminPolicy" -Subjects "IT Admins"
```

---

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

**Scripts:** `Get-PrivilegedRoleReport.ps1`, `Export-EntraAuditLogs.ps1`

**KQL — Failed Sign-ins (last 24 hours):**
```kql
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType != 0
| summarize FailureCount = count() by UserPrincipalName, IPAddress, ResultDescription
| sort by FailureCount desc
```

**KQL — Users Added to Privileged Roles:**
```kql
AuditLogs
| where OperationName == "Add member to role"
| where TargetResources[0].modifiedProperties[0].newValue contains "Admin"
| project TimeGenerated, InitiatedBy, TargetResources
```

**KQL — Sign-ins Outside the US:**
```kql
SigninLogs
| where TimeGenerated > ago(7d)
| where Location !startswith "US"
| project TimeGenerated, UserPrincipalName, Location, IPAddress, ResultType
```

---

## Key Skills Demonstrated

- Hybrid identity architecture and Entra Connect Sync configuration
- Active Directory administration — OUs, groups, GPOs, fine-grained password policies
- PowerShell automation for user lifecycle management (provisioning, onboarding, offboarding)
- Microsoft Graph API integration for identity reporting and auditing
- Role Based Access Control (RBAC) with least privilege principles
- Conditional Access policy design and enforcement
- Privileged Identity Management (PIM) with just-in-time access and approval workflows
- Security monitoring using KQL queries in Azure Log Analytics
- Incident documentation and audit log analysis
