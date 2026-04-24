# How to Configure VLAN - Complete Beginner's Guide

## Question: How do I set up VLANs in Proxmox?

### Answer: Follow these steps

---

## How to Create VLAN

### Step 1: Create Bridge with VLAN
1. Go to **Datacenter** → **Network**
2. Click **Create** → **Linux Bridge**
3. Enter **Name**: vmbr0.100
4. Enter **Bridge ports**: vmbr0.100
5. Set **VLAN ID**: 100

### Step 2: Using CLI
```bash
# Add VLAN interface
ip link add link vmbr0 name vmbr0.100 type vlan id 100

# Or use web interface (easier)
```

---

## How to Assign VLAN to VM

### Step 1: Edit VM Network
1. Select VM → **Hardware** → **Network**
2. Modify VLAN tag: 100

### Step 2: Via CLI
```bash
qm set 100 --net0 virtio,bridge=vmbr0.100
```

---

## Keywords

#vlan #how-to #beginner #network

## Related Articles

- [[VLAN-Config]]
- [[Networking]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
