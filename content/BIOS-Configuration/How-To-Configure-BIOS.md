# How to Configure BIOS/UEFI Settings - Complete Beginner's Guide

## Question: How do I configure BIOS or UEFI for VMs?

### Answer: Configure in VM settings

---

## How to Change BIOS Type

### Step 1: Edit VM
1. Select VM → **Options**
2. Find **BIOS**
3. Select: **SeaBIOS** or **OVMF** (UEFI)

### Step 2: Using CLI
```bash
# SeaBIOS (default)
qm set 100 --bios seabios

# UEFI
qm set 100 --bios ovmf
```

---

## Keywords

#bios #uefi #how-to #beginner #vm

## Related Articles

- [[BIOS-Configuration]]
- [[EFI-Configuration]]
- [[VM]]
- [[Proxmox-VE]]