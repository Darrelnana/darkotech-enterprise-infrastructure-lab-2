# Phase 2 — Network Design: IP Scheme, VLANs, Subnetting

**Status:** ✅ Complete
**Date completed:** March 2026

---

## Objective

Design the complete network architecture for the DarkoTech lab **before creating any VMs**. Committing this to GitHub first means the design survives even if the hypervisor storage is wiped.

---

## Key Design Decision

All lab traffic uses `10.10.x.x` private addressing. The home router runs on `192.168.4.x`, so using a completely different RFC 1918 range eliminates any risk of IP conflicts between lab traffic and home network traffic.

---

## VLAN Summary

| VLAN | Name | Subnet | Hosts |
|---|---|---|---|
| 10 | Management | 10.10.10.0/24 | Proxmox only |
| 20 | Servers | 10.10.20.0/24 | DC01, DC02, Ubuntu-Srv |
| 30 | Workstations | 10.10.30.0/24 | PC01 |
| 40 | Guest | 10.10.40.0/24 | Isolated internet-only |
| 50 | Security | 10.10.50.0/24 | Kali Linux |
| 99 | DMZ | 10.10.99.0/24 | Future public services |

Full details including static IPs, DHCP scopes, firewall rules, and subnetting tables are in:
`/network-config/network-design.md`

---

## CCNA Topics Demonstrated

### Subnetting
- All VLANs use /24 subnets (255.255.255.0)
- 254 usable hosts per subnet
- Network, broadcast, and usable range calculated for each

### VLAN concepts
- VLANs separate broadcast domains logically without requiring separate physical switches
- Each VLAN is a separate Layer 2 network
- Inter-VLAN routing requires a Layer 3 device (pfSense acts as router-on-a-stick)

### 802.1Q Trunking
- `vmbr1` in Proxmox is VLAN-aware, carrying multiple tagged VLANs on one interface
- pfSense sees each VLAN as a separate subinterface
- The Netgear GS308E switch will be configured with trunk port to Proxmox and access ports per VLAN

### Inter-VLAN Routing
- pfSense is the default gateway for all VLANs
- Firewall rules on pfSense control what traffic is permitted between VLANs
- Without a rule permitting it, VLANs cannot communicate — enforced isolation

---

## Security+ Topics Demonstrated

### Defense in Depth
This network design implements multiple security layers:
- **Perimeter**: pfSense firewall separates lab from home network
- **Network segmentation**: VLANs enforce isolation between departments
- **Access control**: Firewall rules enforce least-privilege between segments
- **Isolation**: Kali Linux in VLAN 50 cannot reach internet or other VLANs without explicit permit

### Zero Trust Principles
- No implicit trust between VLANs — everything is blocked by default
- Traffic must be explicitly permitted by firewall rules
- Management VLAN (10) is locked — no other VLAN can reach it

---

## Interview Talking Points

**"Why did you design the network before building it?"**
> Designing first and committing to version control means the design survives hardware failures. In a real enterprise, network design goes through review and approval before any configuration change — this mirrors that process.

**"Why VLANs instead of separate physical networks?"**
> VLANs provide logical separation without requiring separate physical switches for each segment. They're cheaper, more flexible, and industry standard. The security is enforced at the firewall level, not just physical separation.

**"How does inter-VLAN routing work in your lab?"**
> pfSense acts as the router for all VLANs. Each VLAN has pfSense as its default gateway. When a workstation on VLAN 30 needs to reach a server on VLAN 20, the packet goes to pfSense, which checks its firewall rules and either permits or blocks it before routing.
