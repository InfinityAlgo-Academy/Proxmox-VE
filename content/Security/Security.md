---
title: Complete Security Guide
tags:
  - security
  - firewall
  - ssl
  - ssh
  - access
  - guide
---

# Complete Security Guide

## Overview

Harden your Proxmox VE installation with security best practices.

## User Access Control

### Create Users

```bash
# Create user
pveum user add admin@example.com --password "SecurePassword"

# Create read-only user
pveum user add readonly@example.com --password "ReadPassword"

# Disable user
pveum user update admin@example.com --enable 0
```

### User Roles

| Role | Permissions |
|------|------------|
| PVEAudit | Read only |
| PVEUser | Standard |
| PVEPowerAdmin | Power user |
| PVEAdmin | Full admin |

### Permissions

```bash
# Grant VM access
pveum acl modify /vms/ --user admin@example.com --role PVEAdmin

# Grant storage access
pveum acl modify /storage/ --user admin@example.com --role PVEAudit

# Revoke access
pveum acl delete /vms/ --user admin@example.com
```

### Two-Factor Authentication

```bash
# Enable 2FA
pveum user update admin@example.com --2fa 1
```

## SSH Security

### Disable Root Login

```bash
# Edit SSHD config
nano /etc/ssh/sshd_config

# Change
PermitRootLogin no
PasswordAuthentication yes
PubkeyAuthentication yes

# Restart
systemctl restart sshd
```

### SSH Key Authentication

```bash
# Generate key (local)
ssh-keygen -t ed25519

# Add to Proxmox
ssh-copy-id root@192.168.1.100

# Or manually
mkdir /root/.ssh
chmod 700 /root/.ssh
echo "public-key" >> /root/.ssh/authorized_keys
```

### Change SSH Port

```bash
# Edit SSHD config
nano /etc/ssh/sshd_config
Port 2222

# Firewall exception
ufw allow 2222/tcp
```

## Firewall

### UFW Firewall

```bash
# Install
apt install ufw -y

# Default policy
ufw default deny incoming
ufw default allow outgoing

# Allow SSH
ufw allow 2222/tcp

# Allow Proxmox
ufw allow 8006/tcp

# Enable
ufw enable
```

### Proxmox Firewall

```bash
# Enable node firewall
pvenode modify --firewall 1

# Configure datacenter
pvesh create /cluster/firewall/datacenter -enable 1
```

### VM Firewall

```bash
# Enable VM firewall
qm set 100 --net0 virtio,bridge=vmbr0,firewall=1
```

## SSL/TLS Certificates

### Let's Encrypt

```bash
# Install certbot
apt install certbot -y

# Get certificate
certbot certonly --standalone -d proxmox.example.com

# Install
cp /etc/letsencrypt/live/example.com/fullchain.pem \
   /etc/pve/ssl/nginx.pem
cp /etc/letsencrypt/live/example.com/privkey.pem \
   /etc/pve/ssl/nginx.key
```

### Self-Signed Certificate

```bash
# Generate
openssl req -new -newkey rsa:4096 -days 365 -nodes \
  -x509 -subj "/CN=proxmox" \
  -keyout /etc/pve/ssl/nginx.key \
  -out /etc/pve/ssl/nginx.pem
```

## Fail2Ban

### Install & Configure

```bash
# Install
apt install fail2ban -y

# Create config
nano /etc/fail2ban/jail.local

[proxmox]
enabled = true
port = 8006
filter = proxmox
logpath = /var/log/pveproxy/access.log
maxretry = 3
bantime = 3600

# Start
systemctl enable fail2ban
systemctl start fail2ban
```

## Backup Encryption

```bash
# Encrypted backup
vzdump --mode snapshot --vmid 100 \
  --storage local --encrypt 1 --password-file /root/backup.pwd
```

## Network Isolation

### VLAN Security

```bash
# Separate networks
auto vmbr0
iface vmbr0 inet static
    address 192.168.10.100/24

# VMs on isolated VLAN
qm set 100 --net0 virtio,bridge=vmbr0,tag=20
```

### VPN Access

- See [[VPN]]

## Updates & Patching

### Regular Updates

```bash
# Update weekly
apt update && apt full-upgrade -y

# Check for updates
pveam update
```

## Security Checklist

- [ ] Change default root password
- [ ] Disable root SSH login
- [ ] Use SSH keys
- [ ] Enable firewall
- [ ] Enable 2FA
- [ ] Regular backups
- [ ] Update system
- [ ] Use VLANs
- [ ] Enable fail2ban
- [ ] Use VPN

## See Also

- [[index|Back to Proxmox VE]]
- [[Access-Management]]
- [[Firewall-Rules]]
- [[VPN]]
- [[Backup-Strategies]]