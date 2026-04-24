# How to Fix BIOS Boot Issues - Complete Troubleshooting Guide

## Question: My VM won't boot - how do I fix BIOS issues?

### Answer: Systematic troubleshooting

---

## Problem: VM Boots to Black Screen

### Possible Causes
- Boot order wrong
- No bootable disk
- BIOS not configured

### Solution 1: Check Boot Order
```bash
# View current boot order
qm config 100 | grep boot

# Set boot order
qm set 100 --boot order=scsi0
```

### Solution 2: Verify Disk
```bash
# Check disk exists
qm config 100 | grep -E "scsi|sata|ide"

# Add disk if missing
qm set 100 --scsi0 local:32
```

---

## Problem: "No Bootable Device"

### Solution: Configure Boot Device
```bash
# Set boot to disk
qm set 100 --boot order=scsi0

# Enable disk
qm set 100 --delete boot
qm set 100 --boot order=scsi0
```

---

## Problem: DVD/ISO Not Booting

### Solution: Check Boot Order
```bash
# Add ISO
qm set 100 --ide2 local:iso/windows.iso,media=cdrom

# Set boot to CD
qm set 100 --boot order=ide2

# After install, change boot back
qm set 100 --boot order=scsi0
```

---

## Problem: UEFI Boot Fails

### Solution: Reinstall EFI
```bash
# Remove old EFI
qm set 100 --delete efidisk0

# Add new EFI disk
qm set 100 --efidisk0 local:1G

# Set boot order
qm set 100 --boot order=efidisk0
```

---

## Problem: PXE Boot Not Working

### Enable Network Boot
```bash
# Enable network boot
qm set 100 --boot order=net0

# Check network config
qm config 100 | grep net0
```

---

## Problem: Boot Loop

### Solution: Check Boot Files
```bash
# Check disk content
pvesm list local | grep 100

# Verify boot files
ls -la /var/lib/vz/images/100/
```

---

## Keywords

#boot-issues #troubleshooting #bios #how-to #fix

## Related Articles

- [[BIOS-Configuration]]
- [[Troubleshooting]]
- [[Auto-Start]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
