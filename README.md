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
Dell OptiPlex 7000 — Proxmox VE
    │
    ├── vmbr0 (WAN bridge → home network)
    └── vmbr1 (LAN bridge → VLAN-aware)
            │
        pfSense Firewall
            │
    ┌───────┼───────────┬──────────────┬──────────────┐
    │       │           │              │              │
VLAN 20  VLAN 30    VLAN 40        VLAN 50        VLAN 99
Servers  Workstns   Guest          Security        DMZ
```

---

## VM Fleet

| VM | OS | Role | IP | VLAN |
|---|---|---|---|---|
| pfSense | pfSense 2.7 | Firewall, router, DHCP, DNS, VPN | 10.10.10.1 | All |
| DC01 | Windows Server 2022 | Primary domain controller | 10.10.20.10 | 20 |
| DC02 | Windows Server 2022 | Secondary domain controller | 10.10.20.11 | 20 |
| Ubuntu-Srv | Ubuntu 24.04 LTS | Web server, reverse proxy, monitoring | 10.10.20.20 | 20 |
| PC01 | Windows 10/11 | Domain workstation | 10.10.30.10 | 30 |
| Kali | Kali Linux | Attack simulation, pen testing | 10.10.50.10 | 50 |

---

## Phases

| Phase | Title | Status |
|---|---|---|
| 1 | Proxmox + storage foundation | ✅ Complete |
| 2 | Network design — IP scheme, VLANs | ✅ Complete |
| 3 | pfSense — firewall, routing, NAT, VPN | 🔧 In progress |
| 4 | Windows Server — AD, DNS, GPO, PKI | ⏳ Pending |
| 5 | Ubuntu — SSH hardening, NGINX, reverse proxy | ⏳ Pending |
| 6 | Switch configuration — VLANs, trunking, STP | ⏳ Pending |
| 7 | Monitoring — Prometheus, Grafana, syslog | ⏳ Pending |
| 8 | Kali — attack simulation, IR, remediation | ⏳ Pending |
| 9 | Backup & disaster recovery | ⏳ Pending |
| 10 | Portfolio — diagrams, case study, LinkedIn | ⏳ Pending |

---

## Certifications Demonstrated

| Cert | Key topics covered in this lab |
|---|---|
| CompTIA A+ | Hardware, virtualization, OS install/config, remote access, backup/recovery, troubleshooting |
| CCNA | Subnetting, VLANs, trunking, inter-VLAN routing, NAT, DHCP, DNS, ACLs, STP, monitoring |
| Security+ | PKI/CA, VPN/encryption, MFA, defense in depth, incident response, vulnerability assessment, GRC |

---

## Repository Structure

```
/diagrams          — Network diagrams, topology maps
/documentation     — Phase-by-phase build documentation
/network-config    — IP tables, VLAN design, firewall rules
/security-reports  — Vulnerability findings, IR reports
/screenshots       — Evidence of working configurations
/runbooks          — Step-by-step operational procedures
README.md          — This file
```

---

## Skills Demonstrated

- Proxmox VE hypervisor deployment and storage management (LVM-Thin)
- Network design and VLAN segmentation
- pfSense firewall configuration, ACLs, NAT, and VPN
- Active Directory Domain Services, DNS, DHCP, Group Policy
- Role-based access control (RBAC) and least privilege
- Windows Certificate Authority and internal PKI
- Linux server hardening (SSH, UFW, fail2ban)
- NGINX reverse proxy configuration
- Prometheus and Grafana monitoring
- Centralized syslog aggregation
- Penetration testing methodology (Kali Linux)
- Formal incident response lifecycle
- Backup strategy and disaster recovery testing
- Technical documentation and network diagramming
