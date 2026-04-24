# How to Configure LVM Storage - Complete Beginner's Guide

## Question: How do I set up LVM storage?

### Answer: Follow these steps

---

## How to Create LVM Volume Group

### Step 1: Create Physical Volume
```bash
pvcreate /dev/sdb
```

### Step 2: Create Volume Group
```bash
vgcreate vg0 /dev/sdb
```

### Step 3: Create Logical Volume
```bash
lvcreate -L 100G -n vmdata vg0
```

---

## How to Add in Proxmox

### CLI
```bash
pvesm add lvmthin local-lvm --vgname vg0
```

---

## Keywords

#lvm #storage #how-to #beginner

## Related Articles

- [[LVM-Management]]
- [[Storage]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
