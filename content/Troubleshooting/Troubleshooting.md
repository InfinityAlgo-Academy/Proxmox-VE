# Troubleshooting Complete Guide

## Table of Contents

1. [Web UI Issues](#web-ui-issues)
2. [VM Issues](#vm-issues)
3. [Container Issues](#container-issues)
4. [Network Issues](#network-issues)
5. [Storage Issues](#storage-issues)
6. [Cluster Issues](#cluster-issues)
7. [Performance Issues](#performance-issues)
8. [Hardware Issues](#hardware-issues)
9. [Recovery](#recovery)
10. [Logs](#logs)

---

## Web UI Issues

### Port Already in Use

```bash
# Check what's using port 8006
lsof -i :8006
netstat -tlnp | grep 8006

# Find process
ss -tlnp | grep 8006

# Change port if needed
# /etc/default/pveproxy
LISTEN_PORT=8006
```

### SSL Certificate Issues

```bash
# Regenerate certificate
systemctl restart pveproxy

# Check certificate
openssl x509 -in /etc/pve/local/pve-ssl.pem -text -noout

# Use Let's Encrypt
# Datacenter → ACME → Add account
```

### Service Not Running

```bash
# Check status
systemctl status pveproxy
systemctl status pveha-lrm
systemctl status pvedaemon

# Restart services
systemctl restart pveproxy
systemctl restart pvedaemon
```

### High CPU Usage

```bash
# Check processes
top
htop

# Proxmox services
systemctl status pveproxy
```

---

## VM Issues

### VM Won't Start

```bash
# Check logs
qm start 100 2>&1

# Check config
qm config 100

# Check resources
qm list

# Verify disk
pvesm status
```

### No Boot Device

```bash
# Check boot order
qm show boot 100

# Set boot
qm set 100 --boot order=scsi0;ide2

# Or
qm set 100 --boot order=ide2;scsi0
```

### No Disk Space

```bash
# Check storage
pvesm status

# Free space
qm resize 100 rootfs +10G
# Or delete old backups
rm -rf /mnt/backup/dump/*
```

### Slow Performance

```bash
# Enable VirtIO
qm set 100 --scsi0 virtio,...

# Use host CPU
qm set 100 --cpu host

# Enable IO thread
qm set 100 --scsi0 local:...,iothread=1
```

### Live Migration Fails

```bash
# Check resources on target
pvesm status
ssh target free -h

# Check network
ping target-node
```

### Pause/Resume Issues

```bash
# Check memory
qm status 100

# Unpause
qm resume 100

# Or kill stuck VM
qm stop 100 --forceStop 1
```

---

## Container Issues

### Container Won't Start

```bash
# Check logs
pct log 100
pct start 100 2>&1

# Check config
pct config 100

# Check resources
pct list
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

### Resource Limits

```bash
# Check memory
pct exec 100 -- free -h

# Check disk
pct exec 100 -- df -h

# Check CPU
pct exec 100 -- top
```

### Permission Issues

```bash
# Check ownership
ls -la /var/lib/lxc/

# Fix permissions
pct mount 100
chown -R 100000:100000 /var/lib/lxc/100/rootfs
pct umount 100
```

### Nested Container Fails

```bash
# Enable nesting
pct set 100 --features nesting=1

# Check kernel
pct exec 100 -- unshare --user
```

---

## Network Issues

### Bridge Not Working

```bash
# Check bridge
ip link show vmbr0
brctl show vmbr0

# Check eth0
ip link show eth0

# Restart networking
systemctl networking restart
```

### No Internet

```bash
# Check gateway
ip route

# Check DNS
ping 8.8.8.8
ping google.com

# Check NAT
iptables -t nat -L -n
```

### VLAN Issues

```bash
# Check VLAN
ip -d link show vmbr0.100

# List VLANs
cat /proc/net/vlan/vmbr0.100

# Check switch port mode
```

### Bond Issues

```bash
# Check bond
cat /proc/net/bonding/bond0

# Check status
ip -d link show bond0

# Check NICs
ethtool eth0
ethtool eth1
```

### DNS Issues

```bash
# Test DNS
dig proxmox.com
nslookup proxmox.com 8.8.8.8

# Check resolv.conf
cat /etc/resolv.conf
```

---

## Storage Issues

### Storage Not Available

```bash
# Check status
pvesm status

# Check mounts
mount | grep

# Check LVM
lvdisplay
lvs -a

# Check ZFS
zpool status
zfs list
```

### LVM Thin Pool Full

```bash
# Check
lvs -a

# Extend
lvextend -L +50G /dev/pve/vmdata

# Or create new pool
lvcreate -T -L 500G -n vmdata2 pve/vg00
```

### ZFS Issues

```bash
# Check status
zpool status -v
zpool list

# Check health
zpool status

# Import
zpool import
zpool import -d /dev/sda
```

### NFS Mount Fails

```bash
# Test mount
mount -t nfs 192.168.1.50:/mnt/nfs /mnt/test

# Check export
showmount -e 192.168.1.50

# Check firewall
```

### iSCSI Issues

```bash
# Check sessions
iscsiadm -m session

# Check devices
iscsiadm -m node

# Restart
systemctl restart open-iscsi
```

---

## Cluster Issues

### Node Not Joinable

```bash
# Check network
ping node2
ssh node2

# Check firewall
firewall-cmd --list-all

# Check ports
nc -zv node2 5404
nc -zv node2 5405
```

### Quorum Lost

```bash
# Check
pvecm status

# Fix expected votes
pvecm expected 1
```

### Corosync Issues

```bash
# Check
pvecm status
pvecm nodes

# Check logs
pvecm log
journalctl -u corosync

# Restart corosync
systemctl restart corosync
```

### Migration Fails

```bash
# Check resources
ssh target free -h
ssh target pvesm status

# Check network
ssh target hostname
```

---

## Performance Issues

### High CPU

```bash
# Check
top
htop

# KVM
qm perf 100

# Fix
qm set 100 --cpu host
```

### High Memory

```bash
# Check
free -h

# Check balloon
qm status 100

# Disable balloon
qm set 100 --balloon 0
```

### High I/O Wait

```bash
# Check
iostat -x 1

# Check for contention
iotop

# Enable discard
qm set 100 --scsi0 local:...,discard=1
```

### Network Slowness

```bash
# Check traffic
iftop
nethogs

# Use VirtIO
qm set 100 --net0 virtio,...

# Enable multi-queue
qm set 100 --net0 virtio,queues=4
```

---

## Hardware Issues

### CPU Not Supported

```bash
# Check
cat /proc/cpuinfo | grep flags

# Enable nested
modprobe kvm_intel
modprobe kvm_amd

# Check
lsmod | grep kvm
```

### USB Passthrough

```bash
# Check
lsusb

# Find device
lsusb -t

# Bind to VFIO
echo 10de:13c2 > /sys/bus/pci/drivers/vfio-pci/new_id
```

### GPU Passthrough

```bash
# Find GPU
lspci | grep -i vga

# Check IOMMU
dmesg | grep -i iommu
dmesg | grep -E "(DMAR|IOMMU)"

# Enable
# /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
```

---

## Recovery

### Boot to Rescue

```bash
# From boot menu
# Select "Proxmox VE (rescue)"

# Mount
mount -o remount,rw /

# Fix
pvecm updatecerts
```

### Reset Root Password

```bash
# Boot recovery Linux
# Mount root
mount /dev/sda1 /mnt
chroot /mnt
passwd root
exit
reboot
```

### Reinstall Without Losing VMs

```bash
# Backup
vzdump 100 --mode stop --storage local

# Export
qm export 100 raw --output /mnt/backup/vm100.raw

# Reinstall
# Import
qm importdisk 100 /mnt/backup/vm100.raw local
```

### Fix Corrupted Storage

```bash
# LVM
lvscan
lvchange -ay pve/vm-100-disk-0

# ZFS
zpool scrub local-zfs
zpool import
```

---

## Logs

### Important Locations

```bash
# Proxmox logs
/var/log/pveproxy/
/var/log/pve/
/var/log/qemu-server/

# System logs
/var/log/syslog
/var/log/messages
/var/log/kern.log

# Kernel
dmesg

# Cluster logs
pvecm log
```

### Access Logs

```bash
# Proxy access
tail -f /var/log/pveproxy/access.log

# Auth
tail -f /var/log/auth.log

# QEMU
tail -f /var/log/qemu-server/qemu-server.log
```

### Debug Mode

```bash
# Enable debug
systemctl stop pveproxy
pveproxy --debug

# Or start
systemctl start pveproxy@debug
```

---

## Keywords

#troubleshooting #debug #logs #network #storage #cluster #performance #hardware #recovery #rescue

## Links

- [[Proxmox-VE]] - Main documentation
- [[Installation]] - Installation issues
- [[Networking]] - Network issues
- [[Storage]] - Storage issues
- [[VM]] - VM issues
- [[Containers]] - Container issues
- [[Cluster]] - Cluster issues
---

[[index|Back to Proxmox VE]]
