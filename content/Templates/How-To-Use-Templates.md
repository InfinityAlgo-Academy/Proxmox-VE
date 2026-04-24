# How to Use Templates - Complete Beginner's Guide

## Question: How do I create and use VM templates?

### Answer: Follow these steps

---

## How to Create Template

### Step 1: Create VM
Follow normal VM creation steps

### Step 2: Install OS
Install and configure OS

### Step 3: Prepare for Cloning
```bash
# Remove udev rules
rm -f /etc/udev/rules.d/70-*
# Reset hostname
echo "template" > /etc/hostname
```

### Step 4: Convert to Template
```bash
qm template 100
```

---

## How to Clone from Template

### Step 1: Clone VM
1. Select template VM
2. Click **Clone**
3. Set new name and ID

### Step 2: Configure
```bash
qm set 101 --hostname new-vm
qm set 101 --ip 192.168.1.101
```

---

## Keywords

#template #how-to #beginner #clone

## Related Articles

- [[Templates]]
- [[Cloning]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
