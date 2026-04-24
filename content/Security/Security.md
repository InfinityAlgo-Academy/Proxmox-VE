# Security Complete Guide

## Table of Contents

1. [Initial Hardening](#initial-hardening)
2. [User Management](#user-management)
3. [Access Control](#access-control)
4. [Two-Factor Authentication](#two-factor-authentication)
5. [Firewall](#firewall)
6. [Network Security](#network-security)
7. [SSL/TLS](#ssltls)
8. [Backup Security](#backup-security)
9. [VM/Container Isolation](#vmcontainer-isolation)
10. [Updates & Patching](#updates--patching)
11. [Monitoring](#monitoring)
12. [Hardening Checklist](#hardening-checklist)

---

## Initial Hardening

### Change Default Password

```bash
# Change root password
passwd root

# Secure password requirements:
# - Minimum 12 characters
# - Mixed case, numbers, symbols
# - No dictionary words
```

### Create Admin User

```bash
# Create admin user
useradd -m -s /bin/bash admin

# Add to sudo group
usermod -aG sudo admin

# Set password
passwd admin

# Deny root SSH login
# /etc/ssh/sshd_config
PermitRootLogin no
```

### Update System

```bash
# Full update
apt update && apt full-upgrade -y

# Reboot after kernel update
reboot
```

### Disable Unused Services

```bash
# List services
systemctl list-units --type=service

# Disable unnecessary
systemctl disable <service>
systemctl stop <service>
```

---

## User Management

### Add Users

```bash
# Via Web UI
# Permissions → Users → Add

# Via CLI
pveum user add user@pam --password

# LDAP/AD integration
pveum user add user@domain.com --enable true
```

### Role-Based Access

```bash
# Create custom role
pveum role add PVEAuditor -privs "VM.Audit,Datastore.Audit"

# Modify existing role
pveum role modify PVEAdmin -privs "+Datastore.Allocate"

# List roles
pveum role list
```

### Assign Roles

```bash
# User to VM
pveum acl modify /vms/100 --user user@pam --roles PVEAdmin

# User to storage
pveum acl modify /storage/local --user user@pam --roles PVEUser

# User to whole datacenter
pveum acl modify / --user user@pam --roles PVEAuditor
```

### User Permissions

```bash
# Define resource groups
pveum group add developers -members "user1,user2"

# Assign to group
pveum acl modify /vms --group developers --roles PVEUser
```

---

## Access Control

### Resource Permissions

```bash
# Define permissions
# VM permissions
VM.PowerMgmt     - Start, stop, restart
VM.Console      - Access console
VM.Config.Disk  - Configure disks
VM.Config.CPU   - Configure CPU
VM.Config.Memory - Configure memory
VM.Config.Network - Configure network
VM.Snapshot   - Create snapshots
VM.Backup     - Backup VM
VM.Audit      - View VM

# Datastore permissions
Datastore.Allocate    - Create storage
Datastore.Audit     - View storage
Datastore.Modify   - Modify storage

# Node permissions
Node.Allocate   - Create VMs on node
Node.Audit     - View node
Node.Config    - Configure node
```

### API Tokens

```bash
# Create API token
pveum user token add user@pam mytoken

# View tokens
pveum user token list user@pam

# Use token
curl -H "Authorization: PVEAPIToken=user@pam=mytoken" https://pve:8006/api2/json/
```

---

## Two-Factor Authentication

### Enable TOTP

```bash
# Via Web UI
# User → 2FA → Add → TOTP
# Scan QR code with authenticator app

# Or generate manually
# Store backup codes securely
```

### Enable 2FA for Users

```bash
# Require 2FA
pveum user modify user@pam -enable-2fa 1

# Verify
pveum user list
```

---

## Firewall

### UFW Firewall

```bash
# Install
apt install ufw -y

# Enable
ufw enable

# Default policies
ufw default deny incoming
ufw default allow outgoing

# Allow SSH (configure first!)
ufw allow 22/tcp

# Allow Proxmox
ufw allow 8006/tcp comment "Proxmox Web"
ufw allow 5900:5905/tcp comment "VNC"

# Allow cluster ports
ufw allow 5404/udp comment "Corosync"
ufw allow 5405/udp comment "Corosync"

# View rules
ufw status numbered

# Delete rule
ufw delete 1
```

### iptables

```bash
# List rules
iptables -L -n -v
iptables -L -n -v -t nat

# Allow established
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

---

## Network Security

### Change Default Port

```bash
# Edit /etc/default/pveproxy
# Change port
LISTEN_PORT=8006

# Restart
systemctl restart pveproxy
```

### Disable HTTP

```bash
# Ensure SSL only
# /etc/pve/proxy.cfg
```

### VLAN Isolation

```bash
# Separate networks
# Management: VLAN 1
# VM traffic: VLAN 10
# Storage: VLAN 30
# Backup: VLAN 40

# Create isolated bridges
auto vmbr1
iface vmbr1 inet manual
    bridge-ports none
    bridge-stp off
```

### Network Filter

```bash
# Useebtables
ebtables -A FORWARD -s ! aa:bb:cc:dd:ee:ff -j DROP
```

---

## SSL/TLS

### Let's Encrypt

```bash
# Add ACME account
# Web UI: Datacenter → ACME → Add

# Add domain
# Datacenter → ACME → Add Domain

# Enable
# Certificate → ACME → Enable
```

### Custom Certificate

```bash
# Generate CSR
openssl req -new -newkey rsa:4096 -nodes -keyout server.key -out server.csr

# Get certificate
# Or use internal CA

# Install
cp server.crt /etc/pve/local/pve-ssl.pem
cp server.key /etc/pve/local/pve-ssl.key
cp ca.crt /etc/pve/local/pve-ssl.ca

# Restart
systemctl restart pveproxy
```

### Certificate Info

```bash
# View certificate
openssl x509 -in /etc/pve/local/pve-ssl.pem -text -noout

# Check expiration
date -f /etc/pve/local/pve-ssl.pem +"%Y-%m-%d"
```

---

## Backup Security

### PBS Encryption

```bash
# Enable encryption
# PBS → Content → Datastore → Encryption

# Client-side encryption
# Generate key
openssl rand -base64 32 > encryption.key

# Backup with encryption
proxmox-backup-client backup --encryption-key-file encryption.key ...
```

### Encrypted Backups

```bash
# Archive with encryption
tar -czf - . | gpg -c > backup.tar.gz.gpg

# Decrypt
gpg -d backup.tar.gz.gpg | tar -xzf -
```

### Key Management

```bash
# Store key separately
# Never on same server as backup

# Use safe location
# Password manager
# External USB
# HSM
```

---

## VM/Container Isolation

### Resource Limits

```bash
# VM CPU limit
qm set 100 --cpulimit 200

# Container CPU limit
pct set 100 --cpulimit 200

# Memory limit
qm set 100 --memory 4096
pct set 100 --memory 2048

# Network limit
qm set 100 --net0 virtio,bridge=vmbr0,egress=100mbit
```

### Unprivileged Containers

```bash
# Use unprivileged
pct set 100 --unprivileged 1

# With user mapping
lxc.idmap: u 0 100000 65536
lxc.idmap: g 0 100000 65536
```

### AppArmor

```bash
# Check
aa-status

# Enable for container
pct set 100 --apparmor 1
```

---

## Updates & Patching

### Enable Auto-Update

```bash
# Install unattended-upgrades
apt install unattended-upgrades -y

# Configure
dpkg-reconfigure -plow unattended-upgrades
# Select: Yes to automatic reboot
# Select upgrade window
```

### Subscribe to Advisories

```bash
# Check Proxmox no-subscription repo
# /etc/apt/sources.list.d/pve-no-subscription.list

# Update frequently
apt update && apt upgrade -y
```

### Security Updates Only

```bash
# /etc/apt/apt.conf.d/50unattended-upgrades
Unattended-Upgrade::Allowed-Origins {
    "Proxmox":"stable";
};
```

---

## Monitoring

### Log Monitoring

```bash
# View auth logs
tail -f /var/log/auth.log

# Proxmox logs
tail -f /var/log/pveproxy/access.log
grep -i error /var/log/pve/*.log

# System logs
journalctl -xe
```

### Intrusion Detection

```bash
# Install AIDE
apt install aide -y

# Initialize
aideinit

# Update
aide --update

# Check
aide --check
```

### Alerts

```bash
# Setup email alerts
# Datacenter → Options → Email

# Custom scripts
# /etc/pve/admin/Alert.sh
```

---

## Hardening Checklist

### Immediate Actions

- [ ] Change default root password
- [ ] Create admin user, disable root login
- [ ] Enable firewall
- [ ] Change default SSH port
- [ ] Disable root SSH
- [ ] Enable 2FA for admin
- [ ] Use SSL certificates
- [ ] Configure backups
- [ ] Regular updates
- [ ] Monitor logs

### Network Hardening

- [ ] Separate management network
- [ ] Use VLANs
- [ ] Enable firewall rules
- [ ] Use strong passwords
- [ ] Enable SSL/TLS only

### VM/Container Hardening

- [ ] Use unprivileged containers
- [ ] Set resource limits
- [ ] Enable AppArmor
- [ ] Disable unnecessary features
- [ ] Regular patching

### Backup Security

- [ ] Encrypt backups
- [ ] Store keys separately
- [ ] Test restore
- [ ] Offsite storage
- [ ] Regular backups

---

## Keywords

#security #firewall #acl #2fa #encryption #ssl #hardening #updates #monitoring #authentication #permissions #access-control

## Links

- [[Proxmox-VE]] - Main documentation
- [[Backup]] - Backup security
- [[Cluster]] - Cluster security
- [[VM]] - VM security
- [[Containers]] - Container security
- [[Troubleshooting]] - Security issues
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
