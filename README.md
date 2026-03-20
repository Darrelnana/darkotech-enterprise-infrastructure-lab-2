# DarkoTech Enterprise Infrastructure Lab

A fully simulated enterprise IT environment built from scratch on a Dell OptiPlex 7000, designed to demonstrate hands-on skills across CompTIA A+, CCNA, and Security+ certification domains.

---

## Hardware

| Component | Spec |
|---|---|
| Host machine | Dell OptiPlex 7000 |
| CPU | Intel Core i7-8700K @ 3.70GHz (12 threads) |
| RAM | 16GB DDR4 |
| Storage | 476GB NVMe SSD |
| Switch | Netgear GS308E (managed) |
| Hypervisor | Proxmox VE 9.1.6 |

---

## Lab Architecture

```
Internet
    │
Home Router (192.168.4.1)
    │
Dell OptiPlex 7000 — Proxmox VE 9.1.6
    │
    ├── vmbr0 (WAN bridge → home network)
    └── vmbr1 (LAN bridge → VLAN-aware)
            │
        pfSense Firewall (10.10.10.1)
            │
    ┌───────┼──────────┬─────────────┬─────────────┐
    │       │          │             │             │
VLAN 20  VLAN 30   VLAN 40       VLAN 50       VLAN 99
Servers  Workstns  Guest         Security       DMZ
10.10.20 10.10.30  10.10.40      10.10.50      10.10.99
```

---

## VM Fleet

| VM | OS | Role | IP | VLAN |
|---|---|---|---|---|
| pfSense | pfSense CE 2.7.2 | Firewall, router, DHCP, DNS, NAT | 10.10.10.1 | All |
| DC01 | Windows Server 2022 | Primary domain controller, DNS, DHCP | 10.10.20.10 | 20 |
| DC02 | Windows Server 2022 | Secondary domain controller, AD replication | 10.10.20.11 | 20 |
| Ubuntu-Srv | Ubuntu 24.04 LTS | Web server, reverse proxy, monitoring | 10.10.20.20 | 20 |
| PC01 | Windows 11 Pro | Domain workstation | 10.10.30.10 | 30 |
| Kali | Kali Linux | Attack simulation, pen testing | 10.10.50.10 | 50 |

---

## Build Phases

| Phase | Title | Status |
|---|---|---|
| 1 | Proxmox + LVM-Thin storage foundation | ✅ Complete |
| 2 | Network design — IP scheme, VLANs, subnetting | ✅ Complete |
| 3 | pfSense — firewall, VLANs, DHCP, NAT, firewall rules | ✅ Complete |
| 4 | Windows Server — AD, DNS, GPO, RBAC, file shares, DC replication | ✅ Complete |
| 5 | Ubuntu — SSH hardening, NGINX, UFW, fail2ban | ✅ Complete |
| 6 | Switch configuration — VLANs, trunking, STP (Netgear GS308E) | ⏳ Pending |
| 7 | Monitoring — Prometheus, Grafana, alerting, syslog | 🔧 In Progress |
| 8 | Kali Linux — attack simulation, IR lifecycle, remediation | ⏳ Pending |
| 9 | Backup & disaster recovery — snapshots, restore testing | ⏳ Pending |
| 10 | Portfolio — diagrams, case study, risk register, LinkedIn | ⏳ Pending |

---

## Certifications Demonstrated

| Cert | Key topics covered in this lab |
|---|---|
| CompTIA A+ | Hardware, virtualization, OS install/config, remote access, backup/recovery, troubleshooting |
| CCNA | Subnetting, VLANs, trunking, inter-VLAN routing, NAT, DHCP, DNS, ACLs, STP, monitoring |
| Security+ | PKI/CA, VPN/encryption, MFA, defense in depth, incident response, vulnerability assessment, GRC |

---

## Network Design

### VLAN Scheme

| VLAN | Name | Subnet | Gateway | Purpose |
|---|---|---|---|---|
| 10 | Management | 10.10.10.0/24 | 10.10.10.1 | Proxmox host — locked down |
| 20 | Servers | 10.10.20.0/24 | 10.10.20.1 | Domain controllers, Ubuntu |
| 30 | Workstations | 10.10.30.0/24 | 10.10.30.1 | End user machines |
| 40 | Guest | 10.10.40.0/24 | 10.10.40.1 | Isolated internet-only |
| 50 | Security | 10.10.50.0/24 | 10.10.50.1 | Kali Linux — fully isolated |
| 99 | DMZ | 10.10.99.0/24 | 10.10.99.1 | Future public services |

### Firewall Policy Summary

| From | To | Policy |
|---|---|---|
| Servers | Internet | Allow |
| Workstations | Servers | Allow |
| Workstations | Internet | Allow |
| Guest | Internal (10.10.0.0/16) | Block |
| Guest | Internet | Allow |
| Security (Kali) | Anywhere | Block |
| Any | Management VLAN | Block |

---

## Active Directory

| Setting | Value |
|---|---|
| Domain | darkotech.local |
| NetBIOS | DARKOTECH |
| Primary DC | DC01 (10.10.20.10) |
| Secondary DC | DC02 (10.10.20.11) |

### OU Structure
```
darkotech.local
├── _USERS
├── _COMPUTERS
├── _SERVERS
├── _GROUPS
└── _ADMINS
```

### Security Groups
- IT_Admins — Full administrative access
- HR_Users — Access to HR share (H:)
- Finance_Users — Access to Finance share (F:)
- Helpdesk_Team — Tier 1 support access

---

## Repository Structure

```
/diagrams              — Network topology, VLAN diagrams, AD structure
/documentation         — Phase-by-phase build documentation
/network-config        — IP tables, VLAN design, firewall rules
/security-reports      — Vulnerability findings, IR reports (Phase 8)
/screenshots           — Evidence of working configurations
/runbooks              — Step-by-step operational procedures
README.md              — This file
```

---

## Interview One-Liner

"I built a fully segmented enterprise network from scratch — Proxmox hypervisor with LVM-Thin storage, pfSense firewall with 6 VLANs and ACL enforcement, dual Windows domain controllers with Group Policy and RBAC, a hardened Linux web server, Prometheus and Grafana monitoring, and then ran a Kali Linux attack simulation against the whole environment, documented every finding using the CompTIA IR lifecycle, and remediated them — all documented on GitHub."
