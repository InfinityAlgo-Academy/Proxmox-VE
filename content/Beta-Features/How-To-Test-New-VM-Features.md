---
title: How to Test New VM Features - Complete Guide
---

# How to Test New VM Features - Complete Guide

## Question: How do I test new VM features safely?

### Answer: Test in isolated environment

---

## How to Create Test VM

### Step 1: Create Isolated VM
```bash
qm create 999 --name test-vm --memory 4096 --cores 2 --net0 virtio,bridge=vmbr0
```

### Step 2: Use Test Network
```bash
# Create isolated network
auto vmbr-test
iface vmbr-test inet static
    address 10.0.0.1/24
    bridge-ports none
    bridge-stp off
```

---

## How to Test Features

### Step 1: Document Before Test
```bash
qm config 999 > /root/test-baseline.txt
```

### Step 2: Enable and Test
```bash
# Test new feature
qm set 999 --new-feature 1

# Check behavior
qm start 999
qm monitor 999
```

---

## How to Roll Back

### Step 1: Restore Baseline
```bash
qm stop 999
qm destroy 999 --destroy-unlinked

# Recreate from known state
qm restore /backup/test-template 999
```

---

## Keywords

#vm-features #testing #sandbox #how-to

## Related Articles

- [[Beta-Features]]
- [[VM]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
