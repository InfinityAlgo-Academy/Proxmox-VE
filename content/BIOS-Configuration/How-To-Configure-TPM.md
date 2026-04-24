# How to Configure TPM for VMs - Complete Guide

## Question: How do I add TPM to my VM?

### Answer: Virtual TPM configuration

---

## What is TPM?

### Purpose
TPM (Trusted Platform Module) provides:
- Hardware-level security
- Key storage
- Platform integrity
- BitLocker support

---

## How to Add TPM

### Step 1: Check Requirements
- OVMF BIOS required
- TPM 2.0 supported

### Step 2: Add TPM to VM
```bash
# Add TPM device
qm set 100 --tpmstate0 local:1M
```

### Step 3: Enable TPM
```bash
# Enable TPM
qm set 100 --tpm state=present
```

---

## How to Use TPM for BitLocker

### Enable in Windows
1. Install Windows with UEFI
2. Add TPM to VM
3. Enable BitLocker in Windows
4. Recovery key stored in VM

---

## How to Verify TPM

### Check TPM Status
```bash
qm config 100 | grep tpm
```

---

## Keywords

#tpm #security #bitlocker #how-to #hardware

## Related Articles

- [[BIOS-Configuration]]
- [[EFI-Configuration]]
- [[Security]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
