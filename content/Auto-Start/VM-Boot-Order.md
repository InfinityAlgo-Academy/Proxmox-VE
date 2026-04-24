# VM Boot Order Configuration Guide

## Overview

Boot order determines the sequence in which a VM attempts to boot from available devices. Proper boot order configuration is critical for recovery, installation, and production environments.

## Understanding Boot Order

### What is Boot Order

Boot order defines which device the VM tries to boot from first, second, and so on. The VM will attempt each device in sequence until it finds a bootable device.

### Default Boot Order

```bash
# Default is typically:
# scsi0 -> cdrom -> network

# View current boot order
qm config 100 | grep -i boot
```

## Configure Boot Order

### Via Web UI

1. Select VM → **Options**
2. **Boot Order** → Click configured device
3. Enable/disable desired boot devices
4. Drag to reorder

### Via CLI

```bash
# Boot from SCSI first
qm set 100 --boot order=scsi0

# Boot from DVD then disk
qm set 100 --boot order=ide2;scsi0

# Network boot (PXE)
qm set 100 --boot order=net0

# USB boot
qm set 100 --boot order=usb
```

### Multiple Boot Devices

```bash
# Order: DVD, then SCSI, then network
qm set 100 --boot order=ide2;scsi0;net0

# Boot from any available
qm set 100 --boot order=scsi0;net0
```

## Boot Device Options

### Disk Boot (scsi0, sata0, ide0)

```bash
# SCSI disk
qm set 100 --boot order=scsi0

# SATA disk
qm set 100 --boot order=sata0

# IDE disk
qm set 100 --boot order=ide0
```

### DVD/CDROM (ide2)

```bash
# Boot from ISO
qm set 100 --boot order=ide2

# For installation
# 1. Upload ISO
# 2. Attach to VM
# 3. Set boot order ide2 first
```

### Network Boot (net0)

```bash
# PXE boot
qm set 100 --boot order=net0

# For network installation
# Requires DHCP + TFTP server
```

### USB Boot

```bash
# Boot from USB
qm set 100 --boot order=usb
# Requires USB passthrough
```

## Boot Order for Different Scenarios

### Windows Installation

```bash
# 1. Attach Windows ISO
qm set 100 --ide2 local:iso/windows.iso,media=cdrom

# 2. Set boot
qm set 100 --boot order=ide2;scsi0
```

### Linux Installation

```bash
# 1. Attach Linux ISO
qm set 100 --ide2 local:iso/ubuntu-22.04.iso,media=cdrom

# 2. Set boot
qm set 100 --boot order=ide2;scsi0
```

### Recovery Boot

```bash
# Boot to recovery CD
qm set 100 --ide2 local:iso/rescue.iso,media=cdrom

# Set recovery boot
qm set 100 --boot order=ide2;scsi0
```

### Network Install

```bash
# PXE boot only
qm set 100 --boot order=net0

# Configure DHCP/TFTP
```

## Boot Menu

### Enable Boot Menu

```bash
# Enable boot menu (within VM)
# Usually enabled by default
# Press ESC/F2/F12 for menu

# Configure in VM BIOS
qm set 100 --bios ovmf
```

### Boot Menu Hotkeys

| Key | BIOS | UEFI |
|-----|------|------|
| ESC | Boot menu | Boot menu |
| F2 | Setup | Setup |
| F12 | Network boot | Network |

## Troubleshooting Boot Order

### VM Not Booting

```bash
# Check boot order
qm config 100 | grep boot

# Verify boot device exists
qm config 100 | grep -E "scsi|sata|ide|net"

# Check disk status
pvesm status
```

### Boot to Wrong Device

```bash
# Set correct order
qm set 100 --boot order=scsi0;ide2

# Verify after reboot
qm config 100 | grep boot
```

### PXE Boot Not Working

```bash
# Check network
qm config 100 | grep net0

# Verify DHCP
ping 192.168.1.1
```

## Advanced Boot Options

### Boot Custom Kernel

```bash
# Custom boot kernel
qm set 100 --kernel /boot/vmlinuz-custom

# Boot custom initrd
qm set 100 --initrd /boot/initrd.img
```

### EFI Boot

```bash
# Enable EFI
qm set 100 --bios ovmf

# EFI boot order
qm set 100 --efidisk0 local:1G
```

## Related Articles

- [[Auto-Start]]
- [[VM-Configuration]]
- [[BIOS-Configuration]]
- [[EFI-Configuration]]