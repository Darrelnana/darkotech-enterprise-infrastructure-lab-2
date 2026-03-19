# Phase 1 — Proxmox + Storage Foundation

**Status:** ✅ Complete
**Date completed:** March 2026

---

## Objective

Install and configure Proxmox VE as the hypervisor for the DarkoTech lab. Set up storage correctly from the start to support snapshots and avoid the storage issues from the previous build.

---

## Hardware

| Component | Detail |
|---|---|
| Machine | Dell OptiPlex 7000 |
| CPU | Intel Core i7-8700K @ 3.70GHz (6 cores, 12 threads) |
| RAM | 16GB DDR4 |
| Storage | 476GB NVMe SSD (nvme0n1) |
| Secondary | 16GB USB (used for Proxmox installer only) |

---

## Installation Decisions

### Why Proxmox VE?
Proxmox is an enterprise-grade open source hypervisor based on KVM and LXC. It provides a full web UI for VM management, supports LVM-Thin storage with snapshots, and is widely used in real enterprise environments. It directly maps to A+ virtualization topics and is free for lab use.

### Why LVM-Thin over ZFS or standard LVM?
| Option | Snapshots | RAM cost | Chosen |
|---|---|---|---|
| Standard LVM | No | None | No |
| LVM-Thin | Yes | None | **Yes** |
| ZFS | Yes | High (1GB/TB min) | No |

LVM-Thin was chosen because:
- Snapshots are required for the Kali attack simulation phases (take snapshot before attack, restore after)
- Thin provisioning means VMs only consume real disk as they write data — more VMs from same disk
- No RAM overhead unlike ZFS, which would eat into the 16GB available for VMs

### Storage partition layout

| Volume | Size | Purpose |
|---|---|---|
| EFI boot | 1GB | EFI boot partition |
| pve-swap | 8GB | Swap space |
| pve-root | 80GB | Proxmox OS, ISO storage |
| pve/data (LVM-Thin) | ~363GB | All VM disks and snapshots |

`maxroot` was set to 80GB during install and `lvmsize` to 0, leaving all remaining space for the manually created LVM-Thin pool.

---

## Post-Install Configuration

### Repositories fixed
- Disabled `pve-enterprise` repo (requires paid subscription)
- Enabled `pve-no-subscription` repo (free, community supported)
- Disabled Ceph enterprise repo

### Subscription nag disabled
The Proxmox web UI nag popup was disabled for lab use.

### System fully updated
```bash
apt update && apt full-upgrade -y
```

### LVM-Thin pool created
```bash
lvcreate -l 100%FREE -T pve/data
pvesm add lvmthin local-lvm --thinpool pve/data --content images,rootdir
```

---

## Verification

```
pvesm status output:
Name          Type      Status    Total (KiB)   Used (KiB)  Available (KiB)
local         dir       active     81987992       5105744      72671560
local-lvm     lvmthin   active    381239296             0     381239296
```

Both storage pools confirmed active. LVM-Thin pool shows 363GB available for VMs.

---

## Proxmox Network Bridges

| Bridge | Purpose | Config |
|---|---|---|
| vmbr0 | WAN — physical NIC to home router | No VLAN awareness |
| vmbr1 | LAN — internal VM traffic | VLAN aware = Yes |

---

## Lessons Learned from Previous Build

The previous lab was lost when the LVM was deleted to try to reclaim space. Root cause: the Proxmox installer had allocated all disk space to `local-lvm` as standard LVM, leaving no room to grow `pve-root`. 

Fix applied this time: set `maxroot=80` during install and `lvmsize=0`, then manually created the LVM-Thin pool after install. This gives full control over the storage split and uses the superior thin pool format.

---

## A+ Topics Covered

- Virtualization concepts (Type 1 hypervisor, VM resource allocation)
- Storage types and their tradeoffs
- Hardware identification and compatibility
- Operating system installation
- Linux command line basics
- Boot modes (EFI/Secure Boot)

## CCNA Topics Covered

- Network bridge = virtual switch concept
- VLAN-aware bridge configuration
- Network interface types
