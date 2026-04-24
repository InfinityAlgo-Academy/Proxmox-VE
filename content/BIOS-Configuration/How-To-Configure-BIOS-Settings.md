# How to Configure BIOS Settings - Complete Beginner's Guide

## Question: How do I configure BIOS for Proxmox VMs?

### Answer: Configure BIOS type in VM settings

---

## Understanding BIOS Options

### What is BIOS in Virtual Machines?

Just like physical computers, VMs need firmware to start. Proxmox supports two types:

- **SeaBIOS**: Traditional BIOS (legacy, simpler)
- **OVMF**: UEFI BIOS (modern, more features)

---

## How to Change BIOS Type (Web UI)

### Step 1: Edit VM Options
1. Select your VM → **Options** tab
2. Find **BIOS** setting
3. Click dropdown
4. Select: **SeaBIOS** or **OVMF**

### Step 2: Save and Start
Click **OK**, then start the VM

---

## How to Change BIOS (CLI)

### Set SeaBIOS
```bash
# Traditional BIOS
qm set 100 --bios seabios
```

### Set UEFI/OVMF
```bash
# UEFI BIOS
qm set 100 --bios ovmf
```

---

## Which BIOS Should I Choose?

### Use SeaBIOS When:
- Installing old Windows versions (XP, 7)
- Using legacy operating systems
- Maximum compatibility needed
- Faster boot time required

### Use OVMF (UEFI) When:
- Installing Windows 10/11
- Installing modern Linux
- Need secure boot
- Using GPT disks over 2TB
- Need UEFI features

---

## How to Check Current BIOS

### Via CLI
```bash
qm config 100 | grep -i bios
```

### Expected Output
```
bios: ovmf
```

---

## Keywords

#bios #uefi #seabios #vm #how-to #configuration

## Related Articles

- [[BIOS-Configuration]]
- [[EFI-Configuration]]
- [[VM]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
