# DarkoTech Network Design
**Phase 2 — Designed before any VM creation**
**Date:** March 2026

---

## Design Principles

- All lab traffic uses `10.10.x.x` — completely separate from home network (`192.168.4.x`)
- Every device has a static or reserved IP — no guessing
- VLANs enforce department separation — a workstation cannot reach servers unless explicitly permitted
- Kali Linux is fully isolated in its own VLAN — attack simulation cannot escape the lab
- All design decisions committed to GitHub before any VM is created

---

## VLAN Design

| VLAN ID | Name | Subnet | Gateway | Purpose |
|---|---|---|---|---|
| 10 | Management | 10.10.10.0/24 | 10.10.10.1 | Proxmox host management — locked down |
| 20 | Servers | 10.10.20.0/24 | 10.10.20.1 | Domain controllers, Ubuntu server |
| 30 | Workstations | 10.10.30.0/24 | 10.10.30.1 | End user machines, domain clients |
| 40 | Guest | 10.10.40.0/24 | 10.10.40.1 | Isolated guest network |
| 50 | Security | 10.10.50.0/24 | 10.10.50.1 | Kali Linux — attack simulation only |
| 99 | DMZ | 10.10.99.0/24 | 10.10.99.1 | Future public-facing services |

### Why /24 subnets?
Each VLAN gets a full /24 (254 usable hosts). This lab will never have 254 devices per VLAN, but /24 is the industry standard for department-level segmentation and keeps subnetting simple and readable. It maps directly to CCNA subnetting knowledge.

---

## Static IP Assignments

### pfSense Firewall
| Interface | IP Address | Notes |
|---|---|---|
| WAN | 192.168.4.50 | Static on home network |
| LAN / VLAN 10 | 10.10.10.1 | Management gateway |
| VLAN 20 | 10.10.20.1 | Server network gateway |
| VLAN 30 | 10.10.30.1 | Workstation network gateway |
| VLAN 40 | 10.10.40.1 | Guest network gateway |
| VLAN 50 | 10.10.50.1 | Security network gateway |
| VLAN 99 | 10.10.99.1 | DMZ gateway |

### VLAN 20 — Servers (10.10.20.0/24)
| Device | Hostname | IP Address | Role |
|---|---|---|---|
| DC01 | dc01.darkotech.local | 10.10.20.10 | Primary domain controller |
| DC02 | dc02.darkotech.local | 10.10.20.11 | Secondary domain controller |
| Ubuntu-Srv | ubuntu-srv.darkotech.local | 10.10.20.20 | Web server / monitoring |
| DHCP range | — | 10.10.20.100–200 | Dynamic assignments |

### VLAN 30 — Workstations (10.10.30.0/24)
| Device | Hostname | IP Address | Role |
|---|---|---|---|
| PC01 | pc01.darkotech.local | 10.10.30.10 | Windows domain workstation |
| DHCP range | — | 10.10.30.100–200 | Dynamic assignments |

### VLAN 50 — Security (10.10.50.0/24)
| Device | Hostname | IP Address | Role |
|---|---|---|---|
| Kali | kali.darkotech.local | 10.10.50.10 | Attack simulation |
| DHCP range | — | 10.10.50.100–150 | Dynamic assignments |

---

## Proxmox Network Bridges

| Bridge | Type | VLAN Aware | Purpose |
|---|---|---|---|
| vmbr0 | Bridge | No | WAN — physical NIC to home router |
| vmbr1 | Bridge | Yes | LAN — all internal VM traffic, VLAN tagged |

### Why two bridges?
- `vmbr0` carries untagged traffic to the home router — pfSense's WAN interface
- `vmbr1` is VLAN-aware, carrying all internal tagged traffic — pfSense's LAN and all VM interfaces connect here with their VLAN tag

---

## Firewall Rules (pfSense)

| Source | Destination | Action | Reason |
|---|---|---|---|
| VLAN 20 (Servers) | Internet | Allow | Servers need Windows/package updates |
| VLAN 30 (Workstations) | Internet | Allow | Users need internet access |
| VLAN 30 (Workstations) | VLAN 20 (Servers) | Allow | Workstations must reach domain controllers |
| VLAN 20 (Servers) | VLAN 30 (Workstations) | Allow | DC-initiated traffic (Group Policy, etc.) |
| VLAN 40 (Guest) | VLAN 20/30 | Block | Guest isolation — no access to corp network |
| VLAN 40 (Guest) | Internet | Allow | Guests need internet |
| VLAN 50 (Security) | Internet | Block | Kali must not reach internet |
| VLAN 50 (Security) | VLAN 20/30 | Controlled | Only enabled during attack simulation phases |
| Any | VLAN 10 (Management) | Block | Management VLAN locked to admin only |

---

## DHCP Scopes

| VLAN | Scope Range | DNS Server | Lease Time |
|---|---|---|---|
| VLAN 20 | 10.10.20.100–200 | 10.10.20.10 (DC01) | 8 hours |
| VLAN 30 | 10.10.30.100–200 | 10.10.20.10 (DC01) | 8 hours |
| VLAN 40 | 10.10.40.100–200 | 10.10.40.1 (pfSense) | 1 hour |
| VLAN 50 | 10.10.50.100–150 | 10.10.50.1 (pfSense) | 8 hours |

---

## DNS Design

| Zone | Type | Hosted on | Purpose |
|---|---|---|---|
| darkotech.local | Forward | DC01 / DC02 | Internal AD domain resolution |
| 10.10.20.x | Reverse | DC01 | PTR records for server VLAN |
| 10.10.30.x | Reverse | DC01 | PTR records for workstation VLAN |

---

## Defense in Depth Map
*(Security+ — Architecture & Design)*

| Layer | Control | Implementation |
|---|---|---|
| Perimeter | Firewall | pfSense with WAN/LAN separation |
| Network | VLAN segmentation | 6 VLANs with ACL enforcement |
| Host | OS hardening | SSH keys, fail2ban, UFW, Windows hardening |
| Application | Reverse proxy | NGINX with TLS termination |
| Data | Access control | NTFS permissions, RBAC via AD security groups |
| Identity | Authentication | AD Kerberos + MFA (TOTP) |
| Monitoring | Visibility | Prometheus, Grafana, centralized syslog |

---

## Subnetting Reference
*(CCNA topic)*

| VLAN | Network | Subnet Mask | Broadcast | Usable Range | Hosts |
|---|---|---|---|---|---|
| 10 | 10.10.10.0 | 255.255.255.0 (/24) | 10.10.10.255 | 10.10.10.1–254 | 254 |
| 20 | 10.10.20.0 | 255.255.255.0 (/24) | 10.10.20.255 | 10.10.20.1–254 | 254 |
| 30 | 10.10.30.0 | 255.255.255.0 (/24) | 10.10.30.255 | 10.10.30.1–254 | 254 |
| 40 | 10.10.40.0 | 255.255.255.0 (/24) | 10.10.40.255 | 10.10.40.1–254 | 254 |
| 50 | 10.10.50.0 | 255.255.255.0 (/24) | 10.10.50.255 | 10.10.50.1–254 | 254 |
| 99 | 10.10.99.0 | 255.255.255.0 (/24) | 10.10.99.255 | 10.10.99.1–254 | 254 |
