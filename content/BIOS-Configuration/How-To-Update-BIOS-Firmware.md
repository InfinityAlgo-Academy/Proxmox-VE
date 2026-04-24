# How to Update BIOS/UEFI Firmware - Complete Guide

## Question: How do I update VM BIOS firmware?

### Answer: Virtual firmware updates

---

## How to Check BIOS Version

### View Current Version
```bash
# Check BIOS
qm config 100 | grep bios

# OVMF version
qm list 100
```

---

## How to Update OVMF

### Step 1: Update Proxmox
```bash
# Update packages (includes OVMF)
apt update && apt upgrade -y
```

### Step 2: Revert OVMF (if needed)
```bash
# Use specific version
# OVMF is part of ovmf package
apt install ovmf=4.2~beta1-2
```

---

## How to Customize BIOS

### Custom VBIOS
```bash
# Add custom video BIOS
qm set 100 --hostpci0 01:00,pcie=1,romfile=/path/to/vgabios.bin
```

---

## How to Reset BIOS Settings

### Reset to Default
```bash
# Don't work directly
# Recreate VM if issues persist
qm destroy 100
qm create 100 --bios ovmf
```

---

## How to Backup BIOS Settings

### Export Configuration
```bash
# Export VM config
qm config 100 > /backup/vm100-bios-config.txt
```

---

## Keywords

#firmware #update #ovmf #how-to #bios

## Related Articles

- [[BIOS-Configuration]]
- [[Updates]]
- [[VM]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
