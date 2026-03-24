# DarkoTech Network Simulation — Cisco Packet Tracer

This Packet Tracer simulation replicates the DarkoTech enterprise lab network topology, demonstrating CCNA-level networking concepts including VLAN segmentation, 802.1Q trunking, inter-VLAN routing, and Access Control Lists.

---

## File

| File | Description |
|---|---|
| `DarkoTech-Network-Simulation.pkt` | Full network simulation — open with Cisco Packet Tracer 9.0+ |

---

## Topology

```
                    [DarkoTech-Router]
                    Cisco 4331
                    GigabitEthernet0/0/0
                           │
                    (Crossover cable)
                           │
                    [DarkoTech-Switch]
                    Cisco 2960-24TT
                    GigabitEthernet0/1 (Trunk)
                           │
          ┌────────┬────────┬────────┬────────┐
          │        │        │        │        │
        Fa0/2    Fa0/3    Fa0/4    Fa0/5    Fa0/6
        DC01   Ubuntu-Srv  PC01    Kali   Guest-PC
       VLAN 20  VLAN 20  VLAN 30 VLAN 50  VLAN 40
```

---

## Devices

| Device | Model | Role |
|---|---|---|
| DarkoTech-Router | Cisco 4331 | Inter-VLAN router, ACL enforcement |
| DarkoTech-Switch | Cisco 2960-24TT | VLAN switching, 802.1Q trunking |
| DC01 | Server-PT | Simulates Windows domain controller |
| Ubuntu-Srv | Server-PT | Simulates Ubuntu web/monitoring server |
| PC01 | PC-PT | Simulates domain workstation |
| Kali | PC-PT | Simulates attack/security testing machine |
| Guest-PC | PC-PT | Simulates guest network device |

---

## Network Design

### VLANs

| VLAN | Name | Subnet | Gateway | Connected Devices |
|---|---|---|---|---|
| 20 | SERVERS | 10.10.20.0/24 | 10.10.20.1 | DC01, Ubuntu-Srv |
| 30 | WORKSTATIONS | 10.10.30.0/24 | 10.10.30.1 | PC01 |
| 40 | GUEST | 10.10.40.0/24 | 10.10.40.1 | Guest-PC |
| 50 | SECURITY | 10.10.50.0/24 | 10.10.50.1 | Kali |

### IP Addresses

| Device | IP Address | Subnet Mask | Gateway |
|---|---|---|---|
| DC01 | 10.10.20.10 | 255.255.255.0 | 10.10.20.1 |
| Ubuntu-Srv | 10.10.20.20 | 255.255.255.0 | 10.10.20.1 |
| PC01 | 10.10.30.10 | 255.255.255.0 | 10.10.30.1 |
| Kali | 10.10.50.10 | 255.255.255.0 | 10.10.50.1 |
| Guest-PC | 10.10.40.10 | 255.255.255.0 | 10.10.40.1 |

---

## Configurations

### Switch — VLAN and Trunk Configuration

```
vlan 20
 name SERVERS
vlan 30
 name WORKSTATIONS
vlan 40
 name GUEST
vlan 50
 name SECURITY

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20
 description DC01

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 20
 description Ubuntu-Srv

interface FastEthernet0/4
 switchport mode access
 switchport access vlan 30
 description PC01

interface FastEthernet0/5
 switchport mode access
 switchport access vlan 50
 description Kali

interface FastEthernet0/6
 switchport mode access
 switchport access vlan 40
 description Guest-PC

interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 20,30,40,50
 description Trunk-to-Router
```

### Router — Subinterfaces and Inter-VLAN Routing

```
interface GigabitEthernet0/0/0
 no shutdown

interface GigabitEthernet0/0/0.20
 encapsulation dot1Q 20
 ip address 10.10.20.1 255.255.255.0
 description SERVERS-VLAN

interface GigabitEthernet0/0/0.30
 encapsulation dot1Q 30
 ip address 10.10.30.1 255.255.255.0
 description WORKSTATIONS-VLAN

interface GigabitEthernet0/0/0.40
 encapsulation dot1Q 40
 ip address 10.10.40.1 255.255.255.0
 description GUEST-VLAN

interface GigabitEthernet0/0/0.50
 encapsulation dot1Q 50
 ip address 10.10.50.1 255.255.255.0
 description SECURITY-VLAN
```

### Router — Access Control Lists

```
ip access-list extended BLOCK-KALI
 deny ip 10.10.50.0 0.0.0.255 any
 permit ip any any

ip access-list extended BLOCK-GUEST-INTERNAL
 deny ip 10.10.40.0 0.0.0.255 10.10.0.0 0.0.255.255
 permit ip any any

interface GigabitEthernet0/0/0.50
 ip access-group BLOCK-KALI in

interface GigabitEthernet0/0/0.40
 ip access-group BLOCK-GUEST-INTERNAL in
```

---

## Security Policy

| Source | Destination | Action | Reason |
|---|---|---|---|
| VLAN 20 Servers | Anywhere | Permit | Servers need updates and internet |
| VLAN 30 Workstations | VLAN 20 Servers | Permit | Domain authentication, file access |
| VLAN 30 Workstations | Internet | Permit | User internet access |
| VLAN 40 Guest | 10.10.0.0/16 (internal) | **Deny** | Guest isolation — no internal access |
| VLAN 40 Guest | Internet | Permit | Guests can browse internet |
| VLAN 50 Security (Kali) | Anywhere | **Deny** | Kali fully isolated — no access anywhere |

---

## Verification Commands

Run these to verify the configuration is working:

### Switch verification
```
show vlan brief
show interfaces trunk
show interfaces GigabitEthernet0/1 switchport
```

### Router verification
```
show ip interface brief
show ip route
show access-lists
show running-config
```

### Connectivity tests
```
# From PC01 — should succeed
ping 10.10.20.10    (DC01 — inter-VLAN routing)
ping 10.10.20.20    (Ubuntu-Srv — inter-VLAN routing)
ping 10.10.30.1     (own gateway)

# From Kali — should fail (blocked by ACL)
ping 10.10.20.10    (DC01 — blocked)
ping 10.10.30.10    (PC01 — blocked)

# From Guest-PC — DC01 should fail, gateway should work
ping 10.10.20.10    (DC01 — blocked by ACL)
ping 10.10.40.1     (own gateway — should work)
```

---

## CCNA Concepts Demonstrated

| Concept | Where demonstrated |
|---|---|
| VLAN creation and naming | Switch configuration |
| Access port assignment | Fa0/2–Fa0/6 on switch |
| 802.1Q trunk port | Gi0/1 on switch → Gi0/0/0 on router |
| Router-on-a-stick | Router subinterfaces with dot1Q encapsulation |
| Inter-VLAN routing | PC01 pinging DC01 across VLANs |
| Wildcard masks | ACL source/destination matching |
| Named extended ACLs | BLOCK-KALI and BLOCK-GUEST-INTERNAL |
| Inbound ACL application | ip access-group in on subinterfaces |
| Implicit deny | Last rule in every ACL |
| TTL analysis | TTL=127 confirms one routing hop |

---

## How to Open

1. Install **Cisco Packet Tracer 9.0+** (free from netacad.com)
2. Open `DarkoTech-Network-Simulation.pkt`
3. Click on any device → CLI tab to view configurations
4. Use the simulation mode (bottom right) to trace packet flow between devices
