# Proxmox VE Quick Commands Reference

## Table of Contents

1. [VM Commands](#vm-commands)
2. [Container Commands](#container-commands)
3. [Storage Commands](#storage-commands)
4. [Network Commands](#network-commands)
5. [Cluster Commands](#cluster-commands)
6. [System Commands](#system-commands)
7. [Backup Commands](#backup-commands)
8. [User Commands](#user-commands)
9. [API Commands](#api-commands)

---

## VM Commands

### Basic VM Operations

```bash
# List all VMs
qm list

# Create new VM
qm create <vmid> --name "name" --memory <mb> --cores <n>

# Start VM
qm start <vmid>

# Stop VM
qm stop <vmid>

# Restart VM
qm reset <vmid>

# Shutdown (graceful)
qm shutdown <vmid>

# Pause VM
qm pause <vmid>

# Resume VM
qm resume <vmid>
```

### VM Configuration

```bash
# View VM config
qm config <vmid>

# Set memory
qm set <vmid> --memory <mb>

# Set CPU
qm set <vmid> --cores <n>

# Set CPU type
qm set <vmid> --cpu host

# Set boot order
qm set <vmid> --boot order=scsi0;ide2

# Enable/disable
qm set <vmid> --disable 1
qm set <vmid> --disable 0
```

### VM Storage

```bash
# Add disk
qm set <vmid> --scsi0 local:vm-<vmid>-disk-0,size=32G

# Resize disk
qm resize <vmid> scsi0 +10G

# Remove disk
qm set <vmid> --delete scsi0

# Import disk
qm importdisk <vmid> /path/to/image.qcow2 local
```

### VM Network

```bash
# Set network
qm set <vmid> --net0 virtio,bridge=vmbr0

# Change MAC
qm set <vmid> --net0 virtio,bridge=vmbr0,macaddr=XX:XX:XX:XX:XX:XX

# Add second NIC
qm set <vmid> --net1 virtio,bridge=vmbr1
```

### VM Cloning & Snapshots

```bash
# Clone VM
qm clone <vmid> <new_vmid> --full --name "new-name"

# Create snapshot
qm snapshot <vmid> <snapname>

# List snapshots
qm listsnapshot <vmid>

# Revert to snapshot
qm rollback <vmid> <snapname>

# Delete snapshot
qm delsnapshot <vmid> <snapname>
```

### VM Migration

```bash
# Migrate (offline)
qm migrate <vmid> <node>

# Migrate (live)
qm migrate <vmid> <node> --online
```

### VM Console

```bash
# Access console
qm console <vmid>
# Exit: Ctrl+O Q

# View vnc proxy
qm vncproxy <vmid>

# Attach to serial
qm start <vmid> --serial vnc
```

---

## Container Commands

### Basic Container Operations

```bash
# List containers
pct list

# Create container
pct create <ctid> <template> --rootfs local:8

# Start container
pct start <ctid>

# Stop container
pct stop <ctid>

# Restart container
pct restart <ctid>
```

### Container Configuration

```bash
# View config
pct config <ctid>

# Set memory
pct set <ctid> --memory <mb>

# Set CPU
pct set <ctid> --cores <n>

# Set hostname
pct set <ctid> --hostname <name>

# Enable unprivileged
pct set <ctid> --unprivileged 1
```

### Container Networking

```bash
# Set DHCP
pct set <ctid> --net0 name=eth0,bridge=vmbr0,ip=dhcp

# Set static IP
pct set <ctid> --net0 name=eth0,bridge=vmbr0,ip=192.168.1.100/24,gw=192.168.1.1
```

### Container Storage

```bash
# Add mount point
pct set <ctid> --mp0 /path/on/host,mp=/path/in/container

# Resize disk
pct resize <ctid> rootfs +5G
```

### Container Commands Execution

```bash
# Execute command
pct exec <ctid> -- <command>

# Interactive shell
pct exec <ctid> -- bash
```

### Container Cloning

```bash
# Clone container
pct clone <ctid> <new_ctid> --full --hostname <name>
```

---

## Storage Commands

```bash
# List storage
pvesm status

# List storage content
pvesm list <storage>

# Add storage
pvesm add dir <storage> --path /path --content rootdir,images

# Remove storage
pvesm remove <storage>

# Rescan storage
pvesm scan <storage>

# Set storage
pvesm set <storage> --property value
```

---

## Network Commands

```bash
# Show interfaces
ip addr

# Show bridges
bridge link show

# Show bonds
cat /proc/net/bonding/bond0

# Restart network
systemctl networking restart

# Add bridge
# /etc/network/interfaces
# auto vmbr0
# iface vmbr0 inet static
#     address 192.168.1.100
#     netmask 255.255.255.0
#     gateway 192.168.1.1
#     bridge-ports eth0
#     bridge-stp off
```

---

## Cluster Commands

```bash
# Cluster status
pvecm status

# Create cluster
pvecm create <clustername>

# Add node
pvecm add <node-ip>

# Leave cluster
pvecm leave

# Remove node
pvecm delnode <nodename>

# List nodes
pvecm nodes
```

---

## System Commands

```bash
# Update
apt update && apt upgrade -y

# Reboot
reboot

# Shutdown
poweroff

# Check services
systemctl status pveproxy
systemctl status pvedaemon

# View logs
tail -f /var/log/pveproxy/access.log

# Disk usage
df -h

# Memory usage
free -h

# CPU usage
top
```

---

## Backup Commands

```bash
# Backup VM
vzdump <vmid> --mode snapshot --storage local

# Backup container
vzdump <ctid> --mode snapshot --storage local

# List backups
vzdump --list

# Restore VM
qmrestore <backup> <vmid>

# Restore container
pct restore <backup> <ctid>
```

---

## User Commands

```bash
# List users
pveum user list

# Add user
pveum user add <user>@<realm> --password <pass>

# Modify user
pveum user modify <user>@<realm> --enable 1

# Add role
pveum acl modify /path --roles <role> --users <user>
```

---

## API Commands (pvesh)

```bash
# Using pvesh
pvesh get /cluster/status

pvesh get /nodes

pvesh get /cluster/resources

pvesh create /nodes/pve1/qemu --vmid 100 ...

pvesh set /nodes/pve1/qemu/100/config --memory 4096
```

---

## Quick Reference

| Task | Command |
|------|---------|
| List VMs | `qm list` |
| Start VM | `qm start 100` |
| Stop VM | `qm stop 100` |
| Console | `qm console 100` |
| Clone | `qm clone 100 101` |
| List CT | `pct list` |
| Start CT | `pct start 200` |
| Exec | `pct exec 200 -- ls` |
| Status | `pvesm status` |
| Cluster | `pvecm status` |
| Update | `apt update && apt upgrade -y` |

## Keywords

#commands #quick-reference #cli #terminal #bash

## Links

- [[Proxmox-VE]] - Main documentation
- [[VM]] - VM management
- [[Containers]] - Container management
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
