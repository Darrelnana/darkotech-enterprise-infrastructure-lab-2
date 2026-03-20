# Phase 3 — pfSense: Firewall, Routing, VLANs, DHCP, NAT

**Status:** ✅ Complete
**Date completed:** March 2026

---

## Objective

Deploy and configure pfSense as the core firewall and router for the DarkoTech lab. pfSense is the first VM built because every other VM depends on it for routing and VLAN segmentation.

---

## VM Specifications

| Setting | Value |
|---|---|
| VM ID | 100 |
| Name | pfSense |
| OS | pfSense CE 2.7.2 |
| RAM | 1GB |
| Disk | 16GB (local-lvm) |
| vCPU | 1 |
| NIC 1 (net0) | vmbr0 — WAN |
| NIC 2 (net1) | vmbr1 — LAN (VLAN-aware) |

---

## Interface Assignment

| Interface | NIC | IP | Purpose |
|---|---|---|---|
| WAN | vtnet0 | 192.168.4.228 (DHCP from home router) | Internet uplink |
| LAN | vtnet1 | 10.10.10.1/24 | Management network |
| SERVERS | vtnet1.20 | 10.10.20.1/24 | Server VLAN |
| WORKSTATIONS | vtnet1.30 | 10.10.30.1/24 | Workstation VLAN |
| GUEST | vtnet1.40 | 10.10.40.1/24 | Guest VLAN |
| SECURITY | vtnet1.50 | 10.10.50.1/24 | Kali Linux VLAN |
| DMZ | vtnet1.99 | 10.10.99.1/24 | Future public services |

---

## VLAN Configuration

VLANs were created under **Interfaces → Assignments → VLANs** with vtnet1 as the parent interface.

| VLAN ID | Tag | Parent | Description |
|---|---|---|---|
| VLAN 20 | 20 | vtnet1 | Servers |
| VLAN 30 | 30 | vtnet1 | Workstations |
| VLAN 40 | 40 | vtnet1 | Guest |
| VLAN 50 | 50 | vtnet1 | Security |
| VLAN 99 | 99 | vtnet1 | DMZ |

---

## DHCP Scopes

Configured under **Services → DHCP Server:**

| Interface | Range | DNS Server | Lease Time |
|---|---|---|---|
| SERVERS | 10.10.20.100–200 | 10.10.20.10 (DC01) | 8 hours |
| WORKSTATIONS | 10.10.30.100–200 | 10.10.20.10 (DC01) | 8 hours |
| GUEST | 10.10.40.100–200 | 10.10.40.1 (pfSense) | 1 hour |
| SECURITY | 10.10.50.100–150 | 10.10.50.1 (pfSense) | 8 hours |

---

## Firewall Rules

### SERVERS interface
| Action | Source | Destination | Description |
|---|---|---|---|
| Pass | SERVERS subnets | Any | Allow servers to internet |

### WORKSTATIONS interface
| Action | Source | Destination | Description |
|---|---|---|---|
| Pass | WORKSTATIONS subnets | 10.10.20.0/24 | Allow workstations to servers |
| Pass | WORKSTATIONS subnets | Any | Allow workstations to internet |

### GUEST interface
| Action | Source | Destination | Description |
|---|---|---|---|
| Block | GUEST subnets | 10.10.0.0/16 | Block guest to internal network |
| Pass | GUEST subnets | Any | Allow guest internet only |

### SECURITY interface
| Action | Source | Destination | Description |
|---|---|---|---|
| Block | SECURITY subnets | Any | Block Kali internet access |

### WAN interface
| Action | Source | Destination | Description |
|---|---|---|---|
| Pass | Any | WAN:2222 | Allow SSH to Ubuntu-Srv (port forward) |

---

## NAT Configuration

- **Outbound NAT:** Automatic — pfSense auto-generates NAT rules for all internal subnets
- **Port Forward:** WAN:2222 → 10.10.20.20:22 (SSH to Ubuntu-Srv)

---

## CCNA Topics Demonstrated

### Inter-VLAN Routing
pfSense acts as a router-on-a-stick. Each VLAN subinterface (`vtnet1.20`, `vtnet1.30`, etc.) is a separate Layer 3 interface. Traffic between VLANs must pass through pfSense and be explicitly permitted by firewall rules.

### Stateful Firewall vs ACL
pfSense maintains connection state — return traffic for established connections is automatically permitted without needing a reverse rule. This is more secure and efficient than stateless ACLs.

### NAT (PAT — Port Address Translation)
All internal VMs share the single WAN IP (`192.168.4.228`) for outbound internet access. pfSense translates source IPs to the WAN address, maintaining a state table to route return traffic correctly.

### Port Forwarding
Inbound connections to WAN:2222 are redirected to Ubuntu-Srv:22. This requires BOTH a NAT rule AND a firewall rule — the NAT rule redirects the destination, the firewall rule permits the traffic in.

### Rule Order
pfSense evaluates firewall rules top-to-bottom, first match wins. The GUEST Block rule must be above the Allow rule — if reversed, all guest traffic would be permitted before the block rule is reached.

---

## Security+ Topics Demonstrated

### Defense in Depth — Perimeter Layer
pfSense is the perimeter security control. It enforces:
- Network segmentation via VLANs
- Least privilege via explicit permit rules
- Default deny — all traffic is blocked unless explicitly permitted

### Firewall vs NAT — Key Distinction
NAT alone does not provide security. The WAN SSH rule demonstrates this — port forwarding without a matching firewall rule results in dropped packets. Both layers are required.

---

## Interview Talking Points

**"How does inter-VLAN routing work in your lab?"**
> pfSense is the default gateway for all VLANs. When a workstation on VLAN 30 sends traffic to a server on VLAN 20, the packet goes to pfSense at 10.10.30.1, which checks its firewall rules and routes it to 10.10.20.x if permitted.

**"Why did you block Kali from the internet?"**
> A penetration testing VM should never have unrestricted internet access — it could be used to exfiltrate data or download additional attack tools outside a controlled scenario. VLAN 50 has a block-all rule so Kali can only reach internal targets when I explicitly enable that access.

**"What's the difference between a block and a reject rule?"**
> Block silently drops the packet — the sender gets no response and has to wait for timeout. Reject sends back a TCP RST or ICMP unreachable immediately. Block is better for external-facing rules because it gives attackers less information about the network.
