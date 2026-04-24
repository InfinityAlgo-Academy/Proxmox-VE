# How to Configure Cluster - Complete Beginner's Guide

## Question: How do I set up a Proxmox cluster?

### Answer: Follow these steps

---

## How to Create Cluster

### Step 1: First Node
```bash
pvecm create mycluster
```

### Step 2: Add Second Node
```bash
pvecm add 192.168.1.101
```

### Step 3: Verify
```bash
pvecm status
```

---

## How to Enable HA

### Step 1: Add Nodes to HA Group
```bash
ha-manager group add production pve1 pve2 pve3
```

### Step 2: Enable HA for VM
```bash
ha-manager enable vm:100 --group production
```

---

## Keywords

#cluster #how-to #ha #beginner

## Related Articles

- [[Cluster]]
- [[High-Availability]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
