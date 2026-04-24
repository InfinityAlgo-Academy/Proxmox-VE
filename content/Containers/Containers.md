# LXC Containers - Complete Guide

Linux Containers (LXC) in Proxmox VE provide lightweight, efficient virtualization by sharing the host kernel. This guide covers everything from basics to advanced configurations.

## Table of Contents

1. [What is LXC?](#what-is-lxc)
2. [LXC Architecture](#lxc-architecture)
3. [How LXC Works](#how-lxc-works)
4. [Creating LXC Containers](#creating-lxc-containers)
5. [Container Configuration](#container-configuration)
6. [Networking](#networking)
7. [Storage](#storage)
8. [Resource Management](#resource-management)
9. [Unprivileged Containers](#unprivileged-containers)
10. [Security](#security)
11. [Nested Containers](#nested-containers)
12. [Container Templates](#container-templates)
13. [Operations](#operations)
14. [Performance](#performance)
15. [Common Use Cases](#common-use-cases)

---

## What is LXC?

LXC (Linux Containers) is an operating-system-level virtualization method that allows running multiple isolated Linux systems (containers) on a single host.

### Key Characteristics

- **OS-level virtualization**: Shares the host kernel
- **Lightweight**: Minimal overhead (~5MB RAM)
- **Fast**: Boot time < 1 second
- **Efficient**: Near-native performance
- **Linux only**: Cannot run Windows or other OS kernels

### LXC vs LXD vs Docker

| Technology | Purpose | Use Case |
|------------|---------|----------|
| LXC | System containers | Full Linux environments |
| LXD | Container management | Managed LXC via API |
| Docker | Application containers | Microservices, CI/CD |

---

## LXC Architecture

### Namespaces

Namespaces isolate container resources:

```
┌────────────────────────────────────────┐
│           Container 100                 │
├────────────────────────────────────────┤
│  PID Namespace   - Process isolation   │
│  Net Namespace  - Network isolation     │
│  Mount Namespace - Filesystem isolation │
│  User Namespace - User/UID mapping     │
│  UTS Namespace  - Hostname isolation   │
│  IPC Namespace  - IPC isolation        │
└────────────────────────────────────────┘
```

### Control Groups (cgroups)

Resource control and limiting:

```
/sys/fs/cgroup/
├── cpu/
│   └── lxc/
│       └── 100/
│           ├── cpu.cfs_quota_us
│           └── cpu.shares
├── memory/
│   └── lxc/
│       └── 100/
│           ├── memory.limit_in_bytes
│           └── memory.usage_in_bytes
├── blkio/
└── devices/
```

### Security Layers

```
┌─────────────────┐
│  AppArmor/SELinux│
├─────────────────┤
│  Capabilities   │
├─────────────────┤
│  Seccomp        │
├─────────────────┤
│  Namespaces     │
├─────────────────┤
│  cgroups       │
└─────────────────┘
```

---

## How LXC Works

### Container Creation Process

```
1. Allocate container ID (100)
2. Create config in /etc/pve/lxc/
3. Create rootfs directory
4. Extract template
5. Configure network
6. Set resource limits
7. Apply security settings
8. Start container
```

### Container Lifecycle

```
           ┌──────────┐
           │ Created  │
           └────┬─────┘
                │ start
           ┌────▼─────┐
           │ Running  │
           └────┬─────┘
                │ stop
           ┌────▼─────┐
           │ Stopped  │
           └────┬─────┘
                │ delete
           ┌────▼─────┐
           │ Deleted  │
           └──────────┘
```

### File System Structure

```
/var/lib/lxc/
├── 100/
│   ├── config          # Container config
│   ├── rootfs/       # Root filesystem
│   │   ├── bin/
│   │   ├── etc/
│   │   ├── home/
│   │   ├── lib/
│   │   ├── usr/
│   │   └── var/
│   └── fstab         # Mounts
└── 101/
    └── ...
```

---

## Creating LXC Containers

### Prerequisites

1. **Download template**:
```bash
pveam update
pveam list standard
pveam download debian-12 standard 12
```

2. **Check resources**:
```bash
pvesm status
free -h
```

### Method 1: Web UI

1. **Navigate**: CT → Create CT
2. **General Settings**:
   - Hostname: `webserver`
   - Password: `secure-password`
   - Confirm: `secure-password`
3. **Template**:
   - Storage: `local`
   - Template: `debian-12-standard_12.tar.gz`
4. **Root Disk**:
   - Root Disk Size: `8GB`
   - Storage: `local`
5. **CPU**:
   - Cores: `2`
   - Type: `x86-64v2`
6. **Memory**:
   - RAM: `1024MiB`
   - Swap: `512MiB`
7. **Networking**:
   - Network Mode: `Bridged`
   - Bridge: `vmbr0`
   - VLAN Tag: `none`
   - IP Address: `DHCP` (or `192.168.1.100/24`)
   - Gateway: `192.168.1.1`
8. **DNS**:
   - DNS Server: `192.168.1.1`
   - DNS Domain: `local`
9. **Confirm**: Click Create

### Method 2: CLI

```bash
# Create container
pct create 100 \
  local:vztmpl/debian-12-standard_12.tar.gz \
  --rootfs local:8 \
  --memory 2048 \
  --cores 2 \
  --net0 name=eth0,bridge=vmbr0,hwaddr=XX:XX:XX:XX:XX:XX,ip=dhcp \
  --hostname webserver \
  --password your-password \
  --unprivileged 1

# Start container
pct start 100
```

### Method 3: Script

```bash
#!/bin/bash
# create-container.sh

CTID=$1
HOSTNAME=$2
TEMPLATE=$3
STORAGE=$4
ROOTFS=$5
RAM=$6
CORES=$7

pct create ${CTID} \
  ${STORAGE}:vztmpl/${TEMPLATE}.tar.gz \
  --rootfs ${STORAGE}:${ROOTFS} \
  --memory ${RAM} \
  --cores ${CORES} \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --hostname ${HOSTNAME} \
  --password secure-password \
  --unprivileged 1

pct start ${CTID}
```

---

## Container Configuration

### Config File Location

```bash
# Container config
/etc/pve/lxc/100.conf

# Example config
arch: amd64
cores: 2
features: keyctl=1,nesting=1
hostname: webserver
memory: 2048
mp0: /mnt/data,mp=/data,backup=0
net0: name=eth0,bridge=vmbr0,gw=192.168.1.1,hwaddr=BC:24:11:01:01:01,ip=192.168.1.100/24,type=veth
onboot: 1
ostype: debian
rootfs: local:vm-100-disk-0,size=8G
startup: order=1,up=30
swap: 512
unprivileged: 1
```

### Config Options

#### Basic Settings

```bash
# Set hostname
pct set 100 --hostname newhostname

# Set password
pct set 100 --password new-password

# Enable start on boot
pct set 100 --onboot 1

# Startup order
pct set 100 --startup order=1,up=30,down=30
```

#### OS Type

```bash
# Supported OS types
pct set 100 --ostype debian
pct set 100 --ostype ubuntu
pct set 100 --ostype centos
pct set 100 --ostype fedora
pct set 100 --ostype archlinux
pct set 100 --ostype alpine
pct set 100 --ostype gentoo
```

---

## Networking

### Network Types

#### 1. DHCP (Bridged)

```bash
pct set 100 --net0 name=eth0,bridge=vmbr0,type=veth
# Container gets IP from DHCP
```

#### 2. Static IP (Bridged)

```bash
pct set 100 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.100/24,gw=192.168.1.1,type=veth
```

#### 3. NAT (NAT Mode)

```bash
# Create NAT network
# Requires NAT rule on host
iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -j MASQUERADE

# Container uses NAT
pct set 100 --net0 name=eth0,bridge=vmbr0,ip=10.0.0.2/24,gw=10.0.0.1,type=veth
```

#### 4. Host Network

```bash
# Share host network (no isolation)
pct set 100 --net0 name=eth0,bridge=vmbr0,type=phys,eth0
```

### VLAN Configuration

```bash
# VLAN tagging
pct set 100 --net0 name=eth0,bridge=vmbr0,vlan=100,ip=dhcp,type=veth

# Container on VLAN 100 gets DHCP from that VLAN
```

### Multiple Network Interfaces

```bash
# Add second interface
pct set 100 --net1 name=eth1,bridge=vmbr1,ip=dhcp

# Add more interfaces as needed
pct set 100 --net2 name=eth2,bridge=vmbr2,ip=dhcp
```

### Network Firewall

```bash
# Add firewall rules inside container
# Install ufw in container
apt install ufw -y

# Configure
ufw allow 22/tcp   # SSH
ufw allow 80/tcp   # HTTP
ufw allow 443/tcp  # HTTPS
ufw enable
```

### DNS Configuration

```bash
# Set DNS servers
pct set 100 --dns1 8.8.8.8
pct set 100 --dns2 8.8.4.4
pct set 100 --dns domain=local

# Or in config
nameserver 8.8.8.8
nameserver 8.8.4.4
search local
```

---

## Storage

### Root Disk

```bash
# Default root disk
pct set 100 --rootfs local:8G

# Resize root disk
pct resize 100 rootfs +10G
```

### Additional Mounts

```bash
# Bind mount host directory
pct set 100 --mp0 /mnt/host,mp=/mnt/shared

# Bind mount with access mode
pct set 100 --mp0 /mnt/host,mp=/mnt/shared,backup=0

# Mount host file as file
pct set 100 --mp0 /mnt/data.img,mp=/data,size=10G
```

### Device Passthrough

```bash
# Mount block device
# Add to config
lxc.mount.entry: /dev/sda1 mnt/shared none defaults,bind 0 0
```

### Disk quotas

```bash
# Set disk limit
pct set 100 --rootfs local:10G

# Check usage
pct fsck 100
```

---

## Resource Management

### CPU

```bash
# Set CPU cores
pct set 100 --cores 4

# Set CPU limit (percentage)
pct set 100 --cpulimit 200

# Set CPU units (priority)
pct set 100 --cpuunits 1024

# Pin CPU cores
pct set 100 --cores 4 --cpuset 0-3
```

### Memory

```bash
# Set RAM
pct set 100 --memory 4096

# Set Swap
pct set 100 --swap 1024

# Or combined
pct set 100 --memory 4096 --swap 1024
```

### I/O Priority

```bash
# Set I/O weight
pct set 100 --rootfs local:8G,iothread=1

# Or via cgroup
echo 500 > /sys/fs/cgroup/blkio/lxc/100/blkio.weight
```

### Network Bandwidth

```bash
# Limit bandwidth (tc)
# Add to container config
lxc.net.0.token = eth0
lxc.net.0.limit = 1000

# Or use htb qdisc
tc qdisc add dev eth0 root htb rate 100mbit
```

### Resource Monitoring

```bash
# View container resources
pct status 100
pct perf 100

# Check with top
pct exec 100 top

# Check with htop
pct exec 100 htop

# Resource limits file
cat /sys/fs/cgroup/memory/lxc/100/memory.limit_in_bytes
```

---

## Unprivileged Containers

### Requirements

```bash
# Check kernel support
cat /proc/sys/kernel/unprivileged_userns_clone

# Enable if needed
echo 1 > /proc/sys/kernel/unprivileged_userns_clone
```

### User/Group Mapping

```bash
# Add to /etc/pve/lxc/100.conf
lxc.idmap: u 0 100000 65536
lxc.idmap: g 0 100000 65536

# Create mappings
echo "root:100000:65536" >> /etc/subuid
echo "root:100000:65536" >> /etc/subgid

# Restart service
systemctl restart pve-container
```

### Create Unprivileged Container

```bash
# Create with unprivileged flag
pct create 100 local:vztmpl/debian-12.tar.gz --unprivileged 1 --rootfs local:8G

# Or convert existing
pct set 100 --unprivileged 1
```

### Capabilities

```bash
# Drop capabilities
pct set 100 --cap drop net_admin

# Or add specific
# Note: Cannot add capabilities in unprivileged mode
```

### File Permissions

```bash
# Set ownership in container
chown -R 100000:100000 /mnt/data

# Or on host
chown -R 100000:100000 /mnt/host
```

---

## Security

### Security Features

#### 1. AppArmor Profile

```bash
# Create custom profile
cat /etc/pve/lxc/apparmor.d/lxc-100

# Add rules
deny /etc/passwd r,
deny /etc/shadow r,
# Allow specific
allow /var/www/** rwx,

# Apply
pct set 100 --apparmor 1
```

#### 2. Seccomp

```bash
# Enable seccomp
pct set 100 --seccomp 1

# Custom seccomp profile
# Add to config
lxc.seccomp: /var/lib/lxc/seccomp/100.profile
```

#### 3. Keyctl

```bash
# Enable keyctl
pct set 100 --features keyctl=1
```

### Container Isolation

```bash
# Isolate container
pct set 100 --isolation=1

# Restrict device access
# Edit config
lxc.mount.auto: proc:mixed sys:ro mixed
lxc.cap.drop: AUDIT_WRITE NET_ADMIN SYS_ADMIN
```

### Hardening

```bash
# Disable nesting
pct set 100 --features nesting=0

# Disable keyctl
pct set 100 --features keyctl=0

# Disable fuse
pct set 100 --features fuse=0
```

### Monitoring

```bash
# Check security events
pct exec 100 tail -f /var/log/syslog

# View container logs
pct log 100
```

---

## Nested Containers

### Enable Nesting

```bash
# Enable nesting
pct set 100 --features nesting=1

# Verify
pct config 100 | grep nesting
```

### Install Docker in Container

```bash
# Access container
pct console 100

# Install Docker
curl -fsSL https://get.docker.com | sh

# Start Docker
systemctl enable docker
systemctl start docker

# Test
docker run hello-world
```

### Nested Proxmox

```bash
# Enable all features
pct set 100 --features nesting=1,keyctl=1,fuse=1

# Install Proxmox
# Follow standard installation guide
# Note: Nested virtualization limited
```

### LXC in LXC

```bash
# Create inner container
pct create 200 local:vztmpl/alpine.tar.gz
pct start 200

# Test
pct exec 200 -- unshare --user
```

---

## Container Templates

### Official Templates

```bash
# Update template list
pveam update

# List available
pveam list standard
pveam list community

# Download
pveam download debian-12 standard 12
pveam download ubuntu-22.04 standard 22.04
pveam download centos-7 standard 7
pveam download alpine-3.18 standard 3.18
```

### Custom Templates

#### Create from Container

```bash
# Create template
pct template 100

# Verify
ls -la /var/lib/vz/template/cache/
```

#### Manual Template

```bash
# Mount container rootfs
pct mount 100

# Create tarball
tar -czf /var/lib/vz/template/cache/custom.tar.gz -C /var/lib/lxc/100/rootfs .

# Unmount
pct umount 100
```

### Download from Internet

```bash
# Download templates via URL
wget -O /var/lib/vz/template/cache/ubuntu-22.04.tar.gz \
  https://download.proxmox.com/images/ubuntu/ubuntu-22.04-standard.tar.gz
```

### Template Management

```bash
# List templates
pveam list local

# Delete template
rm /var/lib/vz/template/cache/ubuntu-22.04.tar.gz

# Rebuild cache
pveam update
```

---

## Operations

### Container Lifecycle

```bash
# Start
pct start 100

# Stop (graceful)
pct stop 100

# Stop (force)
pct stop 100 --forceStop 1

# Restart
pct restart 100

# Resume (from pause)
pct resume 100

# Pause
pct pause 100
```

### Console Access

```bash
# Attach to console
pct console 100

# Exit: Ctrl+O then Ctrl+Q

# Execute command
pct exec 100 -- ls -la

# Execute with root
pct exec 100 -- su - -c "command"

# SSH directly
ssh root@192.168.1.100
```

### Clone Container

```bash
# Full clone
pct clone 100 101 --full --name clone01

# Linked clone (faster)
pct clone 100 102 --name linked-clone

# Clone with new hostname
pct clone 100 103 --full --hostname newhost
```

### Snapshot

```bash
# Create snapshot
pct snapshot 100 pre-update

# List snapshots
pct listsnapshot 100

# Revert to snapshot
pct rollback 100 pre-update

# Delete snapshot
pct delsnapshot 100 pre-update
```

### Migrate Container

```bash
# Offline migration
pct migrate 100 pve2

# Online migration (live)
pct migrate 100 pve2 --online
```

### Delete Container

```bash
# Stop first
pct stop 100

# Delete
pct destroy 100
```

### Backup/Restore

```bash
# Backup
vzdump 100 --mode stop --storage local --compress lzo

# Restore
# Via web UI: Datacenter → Backup → Restore

# Or CLI
pct restore /mnt/backup/vzdump-lxc-100-2024.tar.gz 100
```

---

## Performance

### Optimization Tips

```bash
# Use SSD storage
# Enable discard for thin provisioning
pct set 100 --rootfs local:8G,discard=1

# Use native network driver
pct set 100 --net0 name=eth0,bridge=vmbr0,type=veth,driver=virtio

# Enablemp
pct set 100 --cores 4 --cpuunits 1024
```

### Benchmark

```bash
# CPU benchmark in container
pct exec 100 -- sysbench cpu --cpu-max-prime=20000 run

# Memory benchmark
pct exec 100 -- sysbench memory --memory-block-size=1M --memory-total-size=10G run

# Disk I/O
pct exec 100 -- dd if=/dev/zero of=test bs=1M count=100
```

### Monitoring

```bash
# Resource usage
pct stats 100

# Historical
pct top 100

# Check cgroup
cat /sys/fs/cgroup/memory/lxc/100/memory.usage_in_bytes
cat /sys/fs/cgroup/cpu/lxc/100/cpuacct.usage
```

### Performance Comparison

| Driver | Network | Disk |
|--------|---------|------|
| virtio (paravirt) | ~10 Gbps | Best |
| veth (bridge) | ~1 Gbps | Good |
|rtl8139 | 100 Mbps | Acceptable |

---

## Common Use Cases

### Web Server

```bash
# Create web server
pct create 100 local:vztmpl/debian-12.tar.gz \
  --rootfs local:10G \
  --cores 2 --memory 2048 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --hostname webserver

# Install web stack
pct exec 100 -- apt install nginx php mariadb-server -y

# Configure
pct exec 100 -- systemctl enable nginx
```

### Database Server

```bash
# Create database container
pct create 101 local:vztmpl/debian-12.tar.gz \
  --rootfs local:20G \
  --cores 4 --memory 8192 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --hostname database

# Install MariaDB
pct exec 101 -- apt install mariadb-server -y
```

### Development Environment

```bash
# Create dev container
pct create 102 local:vztmpl/ubuntu-22.04.tar.gz \
  --rootfs local:30G \
  --cores 4 --memory 8192 \
  --features nesting=1 \
  --hostname devbox

# Install dev tools
pct exec 102 -- apt install git vim python3 docker.io -y
```

### Home Automation

```bash
# Create Home Assistant container
pct create 103 local:vztmpl/debian-12.tar.gz \
  --rootfs local:8G \
  --cores 2 --memory 4096 \
  --hostname homeassistant

# Install HA
pct exec 103 -- apt install python3 python3-pip -y
pct exec 103 -- pip3 install homeassistant
```

### VPN Server

```bash
# Create VPN container
pct create 104 local:vztmpl/debian-12.tar.gz \
  --rootfs local:4G \
  --cores 1 --memory 512 \
  --hostname vpn

# Install WireGuard
pct exec 104 -- apt install wireguard -y
```

---

## Troubleshooting

### Container Won't Start

```bash
# Check logs
pct log 100

# Check config
pct config 100

# Check resources
pvesm status
free -h
```

### No Network

```bash
# Check network config
pct exec 100 -- ip addr

# Check gateway
pct exec 100 -- ip route

# Check DNS
pct exec 100 -- cat /etc/resolv.conf
```

### Resource Issues

```bash
# Check memory
pct exec 100 -- free -h

# Check CPU
pct exec 100 -- top

# Check disk
pct exec 100 -- df -h
```

---

## Keywords

#lxc #container #linux #unprivileged #nesting #template #lxd #docker #namespaces #cgroups #security #network #storage #resource #performance

## Links

- [[Proxmox-VE]] - Main documentation
- [[VM]] - Virtual machines comparison
- [[Storage]] - Container storage
- [[Networking]] - Network configuration
- [[Troubleshooting]] - LXC issues
- [[Security]] - Container security
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
