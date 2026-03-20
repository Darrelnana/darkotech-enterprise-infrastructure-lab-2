# Phase 5 — Ubuntu Server: SSH Hardening, NGINX, fail2ban

**Status:** ✅ Complete
**Date completed:** March 2026

---

## Objective

Deploy a hardened Ubuntu Server providing web hosting, reverse proxy, and log monitoring services. Apply Security+ hardening principles to lock down the server before exposing any services.

---

## VM Specifications

| Setting | Value |
|---|---|
| VM ID | 104 |
| Name | Ubuntu-Srv |
| OS | Ubuntu Server 24.04.4 LTS |
| RAM | 2GB |
| Disk | 50GB (local-lvm) |
| vCPU | 2 |
| IP | 10.10.20.20/24 |
| Gateway | 10.10.20.1 |
| DNS | 8.8.8.8 |
| VLAN | 20 (Servers) |

---

## Packages Installed

| Package | Purpose |
|---|---|
| nginx | Web server and reverse proxy |
| fail2ban | Intrusion prevention — bans IPs after failed attempts |
| ufw | Host-based firewall (Uncomplicated Firewall) |
| curl / wget | HTTP utilities |
| git | Version control |
| net-tools | Network utilities (ifconfig, netstat) |
| prometheus | Metrics collection (Phase 7) |
| prometheus-node-exporter | System metrics exporter |

---

## SSH Hardening

File modified: `/etc/ssh/sshd_config`

| Setting | Value | Security Reason |
|---|---|---|
| PermitRootLogin | no | Root account cannot be targeted directly |
| PasswordAuthentication | no | Key-only auth prevents password brute force |
| X11Forwarding | no | Disables GUI forwarding — unused attack surface |
| MaxAuthTries | 3 | Limits attempts before connection dropped |

### Why key-only authentication?
Password authentication is vulnerable to brute force attacks even with fail2ban. SSH key pairs use asymmetric cryptography — the private key never leaves the client machine, making remote brute force mathematically impractical.

---

## UFW Firewall Rules

Host-based firewall configured to allow only required ports:

| Port | Protocol | Action | Service |
|---|---|---|---|
| 22 | TCP | Allow | SSH |
| 80 | TCP | Allow | HTTP (NGINX) |
| 443 | TCP | Allow | HTTPS (NGINX) |
| All others | Any | Deny | Default deny |

Commands used:
```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

---

## fail2ban Configuration

fail2ban monitors log files and bans IPs that show malicious patterns.

File: `/etc/fail2ban/jail.local`

### SSH jail (`[sshd]`):
| Setting | Value | Meaning |
|---|---|---|
| enabled | true | Actively monitoring SSH |
| maxretry | 3 | Ban after 3 failed attempts |
| bantime | 3600 | Ban for 1 hour (3600 seconds) |
| findtime | 600 | Count attempts within 10-minute window |

### How it works:
1. fail2ban watches `/var/log/auth.log`
2. If 3 failed SSH attempts occur within 10 minutes from the same IP
3. fail2ban inserts an iptables rule blocking that IP for 1 hour
4. Attack is logged and stopped automatically

---

## NGINX Configuration

NGINX is running as a web server and reverse proxy. Default configuration serves a test page on port 80.

Log files monitored:
```
/var/log/nginx/access.log  — all incoming requests
/var/log/nginx/error.log   — errors and warnings
```

Real-time log monitoring:
```bash
sudo tail -f /var/log/nginx/access.log
```

---

## Services Running

| Service | Status | Auto-start |
|---|---|---|
| nginx | Active | Enabled |
| fail2ban | Active | Enabled |
| ufw | Active | Enabled |
| prometheus | Active | Enabled |
| prometheus-node-exporter | Active | Enabled |
| ssh | Active | Enabled |

---

## A+ Topics Demonstrated

- Linux server installation and configuration
- Package management with apt
- Service management with systemctl
- Log file locations and monitoring
- Host-based firewall configuration
- Remote administration via SSH

## Security+ Topics Demonstrated

### Defense in Depth — Host Layer
Multiple overlapping security controls on a single host:
1. **UFW** — controls what ports are accessible
2. **SSH hardening** — restricts how SSH can be used
3. **fail2ban** — actively responds to attack patterns
4. **Key-only auth** — eliminates password-based attack vector

### Hardening Checklist
A formal hardening checklist was followed:
- ✅ Disable root SSH login
- ✅ Disable password authentication
- ✅ Limit max auth attempts
- ✅ Enable host firewall with default deny
- ✅ Install intrusion prevention
- ✅ Enable automatic security updates
- ✅ Remove unnecessary services

### Principle of Least Functionality
Only required ports (22, 80, 443) are open. All other ports are blocked by default — the server cannot be reached on any unexpected port.

---

## Interview Talking Points

**"How did you harden your Linux server?"**
> I followed a hardening checklist — disabled root SSH login, switched to key-only authentication, set a 3-attempt limit before connection drops, enabled UFW with default deny and only opened the three ports the server actually needs, and installed fail2ban to automatically ban IPs that show brute force patterns. I can show you the sshd_config and explain each change.

**"What's fail2ban and why did you use it?"**
> fail2ban is an intrusion prevention tool that monitors log files for attack patterns. When it detects too many failed login attempts from one IP, it inserts a firewall rule blocking that IP automatically. It bridges the gap between static firewall rules and active threat response — it's the difference between a locked door and a door that calls security when someone keeps trying wrong keys.

**"What's the difference between UFW and fail2ban?"**
> UFW is a static firewall — it blocks or allows based on fixed rules you define. fail2ban is dynamic — it adds and removes rules automatically based on observed behavior. You need both: UFW for baseline access control, fail2ban for responding to active attacks.
