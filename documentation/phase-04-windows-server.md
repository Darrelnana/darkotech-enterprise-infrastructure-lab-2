# Phase 4 — Windows Server: Active Directory, DNS, GPO, File Shares

**Status:** ✅ Complete
**Date completed:** March 2026

---

## Objective

Deploy a redundant Windows Server domain infrastructure simulating a real enterprise environment. Build Active Directory with proper OU structure, enforce security policy through Group Policy, implement RBAC through security groups, and configure shared file storage with NTFS permissions.

---

## VM Specifications

### DC01 — Primary Domain Controller
| Setting | Value |
|---|---|
| VM ID | 102 |
| Name | DC01 |
| OS | Windows Server 2022 Standard (Desktop Experience) |
| RAM | 3GB |
| Disk | 60GB (local-lvm) |
| vCPU | 2 |
| IP | 10.10.20.10/24 |
| Gateway | 10.10.20.1 |
| DNS | 127.0.0.1 (self) |
| VLAN | 20 (Servers) |

### DC02 — Secondary Domain Controller
| Setting | Value |
|---|---|
| VM ID | 103 |
| Name | DC02 |
| OS | Windows Server 2022 Standard (Desktop Experience) |
| RAM | 2GB |
| Disk | 40GB (local-lvm) |
| vCPU | 2 |
| IP | 10.10.20.11/24 |
| Gateway | 10.10.20.1 |
| DNS | 10.10.20.10 (DC01) |
| VLAN | 20 (Servers) |

### PC01 — Domain Workstation
| Setting | Value |
|---|---|
| VM ID | 101 |
| Name | PC01 |
| OS | Windows 11 Pro |
| RAM | 4GB |
| Disk | 40GB (local-lvm) |
| vCPU | 2 |
| IP | 10.10.30.10/24 |
| Gateway | 10.10.30.1 |
| DNS | 10.10.20.10 (DC01) |
| VLAN | 30 (Workstations) |

---

## Active Directory Domain

| Setting | Value |
|---|---|
| Domain name | darkotech.local |
| NetBIOS name | DARKOTECH |
| Forest functional level | Windows Server 2016 |
| Domain functional level | Windows Server 2016 |
| Primary DC | DC01 (10.10.20.10) |
| Secondary DC | DC02 (10.10.20.11) |

---

## Organizational Unit Structure

```
darkotech.local
├── _USERS          — Standard domain user accounts
├── _COMPUTERS      — Domain-joined workstations
├── _SERVERS        — Domain-joined servers
├── _GROUPS         — Security groups
└── _ADMINS         — Administrative accounts
```

Underscore prefix keeps custom OUs at the top of the list, separate from default AD containers.

---

## Security Groups

| Group | OU | Type | Scope | Members |
|---|---|---|---|---|
| IT_Admins | _GROUPS | Security | Global | mjohnson |
| HR_Users | _GROUPS | Security | Global | jdoe |
| Finance_Users | _GROUPS | Security | Global | jsmith |
| Helpdesk_Team | _GROUPS | Security | Global | — |

---

## User Accounts

| Name | Username | OU | Group | Role |
|---|---|---|---|---|
| John Smith | jsmith | _USERS | Finance_Users | Finance department |
| Jane Doe | jdoe | _USERS | HR_Users | HR department |
| Mike Johnson | mjohnson | _USERS | IT_Admins | IT administrator |

---

## Group Policy Objects

### DarkoTech Security Policy (linked to darkotech.local)

**Password Policy** (`Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy`):

| Setting | Value | Reason |
|---|---|---|
| Minimum password length | 12 characters | Industry standard minimum |
| Password complexity | Enabled | Requires mixed character types |
| Maximum password age | 90 days | Forces regular rotation |
| Minimum password age | 1 day | Prevents immediate reuse |
| Password history | 10 passwords | Prevents cycling back |

**Account Lockout Policy** (`Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Account Lockout Policy`):

| Setting | Value | Reason |
|---|---|---|
| Lockout threshold | 5 attempts | Blocks brute force |
| Lockout duration | 30 minutes | Auto-unlocks after 30 min |
| Reset counter after | 30 minutes | Resets failed attempt count |

**Drive Mappings** (`User Configuration → Preferences → Windows Settings → Drive Maps`):

| Drive | UNC Path | Label | Access |
|---|---|---|---|
| H: | \\DC01\HR | HR Share | HR_Users |
| F: | \\DC01\Finance | Finance Share | Finance_Users |
| I: | \\DC01\IT | IT Share | IT_Admins |

---

## Shared Folders and NTFS Permissions

Folder structure on DC01:
```
C:\Shares\
├── HR\
├── Finance\
└── IT\
```

### NTFS Permissions (Least Privilege)

| Folder | Group | NTFS Permission | Share Permission |
|---|---|---|---|
| HR | HR_Users | Modify | Full Control |
| HR | IT_Admins | Full Control | Full Control |
| Finance | Finance_Users | Modify | Full Control |
| Finance | IT_Admins | Full Control | Full Control |
| IT | IT_Admins | Full Control | Full Control |

**Everyone** was removed from all folders. Only named security groups have access.

---

## AD Replication (DC01 ↔ DC02)

DC02 was promoted as an additional domain controller to `darkotech.local`. Replication was verified with:

```
repadmin /replsummary
```

Both domain controllers maintain a full copy of AD, DNS zones, and SYSVOL. If DC01 fails, DC02 continues authenticating users with no service interruption.

---

## DNS Configuration

DC01 hosts the primary DNS zone for `darkotech.local`. All domain-joined machines point to DC01 (10.10.20.10) as their DNS server.

| Zone | Type | Records |
|---|---|---|
| darkotech.local | Primary Forward | A records for all servers |
| 20.10.10.in-addr.arpa | Primary Reverse | PTR records for VLAN 20 |

---

## A+ Topics Demonstrated

- Windows Server 2022 installation and initial configuration
- Active Directory Domain Services role installation
- Domain controller promotion
- User and group management
- Group Policy creation and linking
- Shared folder creation and permissions
- Remote Desktop Protocol (RDP) access
- Windows Event Viewer for audit logging

## CCNA Topics Demonstrated

- DNS A records and PTR records
- DHCP integration with AD
- Domain controller redundancy
- Name resolution within the lab network

## Security+ Topics Demonstrated

- Role-Based Access Control (RBAC) — users get access based on group membership, not individual assignment
- Least privilege — no user gets more than their role requires
- Password policy enforcement via GPO
- Account lockout policy — brute force prevention
- Separation of duties — HR, Finance, IT have separate shares with no cross-access

---

## Interview Talking Points

**"Explain RBAC and how you implemented it."**
> I created security groups for each department — HR_Users, Finance_Users, IT_Admins. Users are assigned to groups, and groups are assigned to resources. No individual user permissions exist. If someone changes roles, I just move their group membership — one change updates all their permissions automatically.

**"Why did you deploy a second domain controller?"**
> Single point of failure. If DC01 goes down with only one DC, no one can log in — Kerberos authentication requires a domain controller. DC02 holds a full replica of AD and automatically takes over. It also distributes authentication load. This is standard practice in every enterprise environment.

**"What's the difference between NTFS and share permissions?"**
> Share permissions apply when accessing over the network. NTFS permissions apply both locally and over the network. When both exist, the most restrictive combination wins. Best practice is to set share permissions to Full Control for the security group and use NTFS to enforce the actual restrictions — that way all access control logic lives in one place.
