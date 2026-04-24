---
title: How to Configure Boot Order - Complete Guide
---

# How to Configure Boot Order - Complete Guide

## Question: How do I set the boot order for my VM?

### Answer: Configure boot priority in VM settings

---

## Understanding Boot Order

### What is Boot Order?
Boot order determines which device your VM tries to boot from first. Common order:
1. DVD/CD (installation)
2. Hard Disk (normal boot)
3. Network (PXE boot)
4. USB

---

## How to Configure Boot Order (Web UI)

### Step 1: Edit VM Options
1. Select VM → **Options** tab
2. Find **Boot Order**
3. Click configure button
4. Check devices to enable
5. Drag to reorder

### Step 2: Save
Click **OK**

---

## How to Configure Boot Order (CLI)

### Boot from Disk First
```bash
qm set 100 --boot order=scsi0
```

### Boot from DVD, then Disk
```bash
qm set 100 --boot order=ide2;scsi0
```

### Boot from Network
```bash
qm set 100 --boot order=net0
```

### Multiple Devices
```bash
qm set 100 --boot order=ide2;scsi0;net0
```

---

## Boot Order Examples

### Windows Installation
```bash
# 1. Attach ISO
qm set 100 --ide2 local:iso/windows.iso,media=cdrom

# 2. Boot from ISO
qm set 100 --boot order=ide2

# 3. After install, change boot to disk
qm set 100 --boot order=scsi0
```

### Linux Installation
```bash
# Boot from ISO
qm set 100 --boot order=ide2;scsi0
```

### Network Install
```bash
# PXE boot only
qm set 100 --boot order=net0
```

---

## Keywords

#boot-order #boot-priority #how-to #configuration

## Related Articles

- [[BIOS-Configuration]]
- [[Auto-Start]]
- [[VM]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
