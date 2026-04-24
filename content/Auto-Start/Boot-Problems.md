---
title: Boot Problems Troubleshooting Guide
---

# Boot Problems Troubleshooting Guide

## Overview

Comprehensive troubleshooting guide for VM boot problems. Learn to identify and fix boot failures quickly.

## Common Boot Errors

### Error: No Bootable Device

**Symptoms:**
- Black screen with "No bootable device"
- VM stops at BIOS/firmware screen

**Solutions:**
```bash
# 1. Check boot order
qm config 100 | grep boot

# 2. Verify disk exists
qm config 100 | grep -E "scsi|sata|ide"

# 3. Set boot order
qm set 100 --boot order=scsi0
```

### Error: Boot Disk Not Found

**Symptoms:**
- "Boot disk not found"
- BIOS reports no bootable media

**Solutions:**
```bash
# 1. Check storage
pvesm status

# 2. Verify disk is attached
qm config 100 | grep scsi0

# 3. Check disk content
pvesm list local | grep 100

# 4. Recreate disk if needed
qm set 100 --scsi0 local:32
```

### Error: PXE Boot Failed

**Symptoms:**
- Network boot attempts but fails

**Solutions:**
```bash
# 1. Check network
qm config 100 | grep net0

# 2. Verify DHCP
# Check DHCP server is running

# 3. Configure PXE
# Set boot order: net0
qm set 100 --boot order=net0
```

### Error: Boot Loop

**Symptoms:**
- VM constantly reboots
- Never reaches OS

**Solutions:**
```bash
# 1. Check boot logs
qm console 100

# 2. Remove bootable media
qm set 100 --ide2 none

# 3. Verify disk is bootable
```

## Boot Order Issues

### Wrong Boot Device First

```bash
# Check current order
qm config 100 | grep boot

# Set correct order
qm set 100 --boot order=scsi0;ide2

# Verify
qm config 100 | grep boot
```

### DVD Boot Without ISO

```bash
# Remove empty DVD
qm set 100 --ide2 none

# Set boot to disk
qm set 100 --boot order=scsi0
```

## Storage Boot Issues

### Storage Not Ready

```bash
# Check storage status
pvesm status

# Check mount
mount | grep /var/lib/vz

# Restart storage
systemctl restart pve-storage
```

### Slow Storage

```bash
# Add boot delay
qm set 100 --startup-delay 10
```

## Network Boot Issues

### DHCP Not Responding

```bash
# Check DHCP service
systemctl status isc-dhcp-server

# Check network
qm config 100 | grep net0

# Verify switch port
```

### TFTP Server Down

```bash
# Check TFTP
systemctl status tftp-hpa

# Start TFTP
systemctl start tftp-hpa
```

## Emergency Boot Recovery

### Boot to Recovery ISO

```bash
# 1. Upload recovery ISO
# 2. Attach ISO
qm set 100 --ide2 local:iso/rescuecd.iso,media=cdrom

# 3. Set boot
qm set 100 --boot order=ide2;scsi0

# 4. Start VM
qm start 100
```

### Fix Partition Table

```bash
# 1. Boot to Linux rescue
# 2. Run fsck
fsck /dev/sda1

# 3. Fix GPT
gdisk /dev/sda
```

## Boot Recovery Commands

### Common Fixes

```bash
# Reset boot configuration
qm set 100 --boot order=scsi0

# Reset EFI
qm set 100 --efidisk0 local:1G,efitype=4g

# Reset BIOS
qm set 100 --bios seabios
```

## Related Articles

- [[Auto-Start]]
- [[VM-Boot-Order]]
- [[Troubleshooting]]
- [[Storage-Configuration]]
---

[[index|Back to Proxmox VE]]
