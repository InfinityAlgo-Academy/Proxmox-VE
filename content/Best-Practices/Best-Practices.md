# Proxmox VE Best Practices Guide

## Table of Contents

1. [Initial Setup Best Practices](#initial-setup-best-practices)
2. [Security Best Practices](#security-best-practices)
3. [Performance Best Practices](#performance-best-practices)
4. [Backup Best Practices](#backup-best-practices)
5. [Network Best Practices](#network-best-practices)
6. [Storage Best Practices](#storage-best-practices)
7. [Maintenance Best Practices](#maintenance-best-practices)
8. [Cluster Best Practices](#cluster-best-practices)
9. [Monitoring Best Practices](#monitoring-best-practices)
10. [Disaster Recovery](#disaster-recovery)

---

## Initial Setup Best Practices

### First Steps After Install

```bash
# 1. Update system immediately
apt update && apt full-upgrade -y
reboot

# 2. Configure repository
sed -i 's/^deb/#deb/' /etc/apt/sources.list.d/pve-enterprise.list
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-community.list
apt update

# 3. Set timezone
timedatectl set-timezone America/New_York

# 4. Configure NTP
systemctl enable systemd-timesyncd
timedatectl set-ntp true
```

### Hostname & DNS

```bash
# Set proper hostname
hostnamectl set-hostname pve1.yourdomain.local

# Configure DNS
cat > /etc/resolv.conf << EOF
nameserver 1.1.1.1
nameserver 1.0.0.1
search local
EOF
```

### Essential Packages

```bash
# Install useful tools
apt install -y curl wget git htop iftop iotop net-tools tcpdump \
  nfs-common cifs-utils rsync chrony fail2ban ufw
```

---

## Security Best Practices

### User Management

```bash
# Create separate admin user
useradd -m -s /bin/bash admin
usermod -aG sudo admin
passwd admin

# Disable root login
sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
systemctl restart sshd

# Use key-based SSH
mkdir -p /root/.ssh
chmod 700 /root/.ssh
```

### Firewall Configuration

```bash
# UFW basic setup
apt install ufw -y

# Default policies
ufw default deny incoming
ufw default allow outgoing

# Allow Proxmox
ufw allow 8006/tcp comment "Proxmox Web"
ufw allow 22/tcp comment "SSH"

# Enable firewall
ufw enable
```

### Two-Factor Authentication

```bash
# Enable 2FA via Web UI
# Datacenter → Users → 2FA → Add → TOTP
# Or use Authelia
```

### SSL Certificates

```bash
# Use Let's Encrypt
# Web UI: Datacenter → ACME → Enable

# Or manually
# Replace certs in /etc/pve/local/
```

---

## Performance Best Practices

### CPU Settings

```bash
# Enable CPU governor
echo performance > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
echo performance > /sys/devices/system/cpu/cpu1/cpufreq/scaling_governor

# For all cores
for cpu in /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor; do
    echo performance > $cpu
done
```

### Memory Management

```bash
# Tune memory
cat >> /etc/sysctl.conf << EOF
vm.swappiness=10
vm.min_free_kbytes=65536
EOF

sysctl -p
```

### Network Optimization

```bash
# Network tuning
cat >> /etc/sysctl.conf << EOF
net.core.rmem_max=16777216
net.core.wmem_max=16777216
net.ipv4.tcp_rmem=4096 87380 16777216
net.ipv4.tcp_wmem=4096 87380 16777216
net.ipv4.tcp_congestion_control=bbr
EOF

sysctl -p
```

### Disk I/O Scheduler

```bash
# Set I/O scheduler
echo none > /sys/block/sda/queue/scheduler
# Or for SSD
echo deadline > /sys/block/sda/queue/scheduler
```

---

## Backup Best Practices

### 3-2-1 Backup Rule

| Type | Description |
|------|-------------|
| 3 | Three copies of data |
| 2 | Two different storage types |
| 1 | One copy offsite |

### Backup Configuration

```bash
# Create backup job
# Datacenter → Backup → Add

# Schedule: Daily at 2 AM
# Storage: local-backup
# Compression: lzo
# Mode: Snapshot
# Retention: 7 daily, 4 weekly, 3 monthly
```

### Offsite Backup

```bash
# rsync to remote
rsync -avz --delete /mnt/backup/dump/ user@backup-server:/backup/

# Or use rclone
rclone sync /mnt/backup/dump backup-remote:ProxmoxBackups
```

---

## Network Best Practices

### Network Segmentation

| Network | VLAN | Purpose |
|---------|------|---------|
| Management | 1 | Proxmox management |
| VM Network | 10 | Regular VMs |
| Storage | 30 | NFS/iSCSI storage |
| Guest | 20 | Untrusted VMs |

### Bond Configuration

```bash
# Use LACP (802.3ad) for production
# Active-backup for home lab
auto bond0
iface bond0 inet static
    bond-slaves eth0 eth1
    bond-mode 802.3ad
    bond-miimon 100
    bond-xmit-hash-policy layer2+3
```

### DNS Configuration

```bash
# Use Pi-hole or AdGuard
# Set as primary DNS for all VMs
# Use 1.1.1.1 / 8.8.8.8 as secondary
```

---

## Storage Best Practices

### Storage Type Selection

| Use Case | Storage Type |
|---------|-------------|
| Boot disks | ZFS Mirror |
| VM disks | LVM-Thin or ZFS |
| Backups | NFS |
| ISO/Template | Directory |
| Fast VM cache | NVMe |

### ZFS Best Practices

```bash
# Create mirror for boot pool
zpool create -f mirror boot-pool mirror /dev/sda /dev/sdb

# Set properties
zfs set compression=lz4 boot-pool
zfs set atime=off boot-pool
zfs set checksum=sha256 boot-pool
```

### NFS Best Practices

```bash
# Use dedicated network
# Mount with hard,intr options
mount -t nfs -o hard,intr,rsize=8192,wsize=8192 192.168.1.50:/data /mnt/data

# Add to /etc/fstab
192.168.1.50:/data /mnt/data nfs hard,intr,rsize=8192,wsize=8192 0 0
```

---

## Maintenance Best Practices

### Regular Tasks

| Task | Frequency |
|------|-----------|
| System update | Weekly |
| Health check | Daily |
| Backup verify | Weekly |
| Log review | Weekly |
| Storage check | Daily |
| Reboot | Monthly |

### Update Procedure

```bash
# Update Proxmox
apt update && apt upgrade -y
reboot

# Update all containers
for ct in $(pct list | awk 'NR>1 {print $1}'); do
    pct exec $ct -- bash -c "apt update && apt upgrade -y"
done
```

### Log Rotation

```bash
# Configure logrotate
cat > /etc/logrotate.d/pve << EOF
/var/log/pve*.log {
    daily
    rotate 7
    compress
    delaycompress
    notifempty
    create 0644 root root
}
EOF
```

---

## Cluster Best Practices

### Quorum Configuration

```bash
# Ensure odd number of nodes
# Minimum 3 nodes for production
# Use redundant network links
```

### Cluster Network

```bash
# Use dedicated network
# Minimum 1GbE
# 10GbE recommended
```

### HA Best Practices

```bash
# Enable HA for critical VMs
pct set VMID --ha 1
qm set VMID --ha 1

# Configure fencing
# Use IPMI or iLO
```

---

## Monitoring Best Practices

### Essential Metrics

```bash
# Monitor:
# - CPU usage
# - Memory usage  
# - Storage capacity
# - Network traffic
# - VM/CT status
# - Service health
```

### Alerting

```bash
# Set up notifications
# Datacenter → Options → Email

# Use Grafana for dashboards
# Set up Prometheus for metrics
```

### Health Checks

```bash
#!/bin/bash
# health-check.sh

# Check node
uptime

# Check services
for svc in pveproxy pvedaemon corosync; do
    systemctl is-active $svc
done

# Check storage
pvesm status

# Check VMs
qm list
pct list
```

---

## Disaster Recovery

### Recovery Plan

1. **Document configuration**
2. **Regular backups**
3. **Offsite copies**
4. **Recovery procedures**
5. **Test restoration**

### Backup Critical Data

```bash
# Export:
# - VM configurations
# - Network config
# - Storage config
# - User permissions

# Commands:
qm config 100 > vm-100-config.txt
pct config 200 > ct-200-config.txt
cat /etc/network/interfaces > network-config.txt
```

### Recovery Procedures

```bash
# Boot to rescue mode
# Restore from backup
# Reconfigure network
```

---

## Quick Reference Checklist

### Initial Setup
- [ ] Update system
- [ ] Configure repo
- [ ] Set hostname/DNS
- [ ] Install tools
- [ ] Configure firewall
- [ ] Enable 2FA

### Performance
- [ ] CPU governor
- [ ] Memory tuning
- [ ] Network tuning
- [ ] Disk scheduler

### Backup
- [ ] Configure backup job
- [ ] Test restore
- [ ] Offsite backup

### Maintenance
- [ ] Set update schedule
- [ ] Set monitoring
- [ ] Configure alerts
- [ ] Document recovery

## Keywords

#best-practices #security #performance #backup #maintenance #hardening #optimization

## Links

- [[Proxmox-VE]] - Main documentation
- [[Security]] - Security hardening
- [[Installation]] - Installation guide
- [[Troubleshooting]] - Issues
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
