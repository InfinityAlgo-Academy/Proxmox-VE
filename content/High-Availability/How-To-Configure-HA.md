---
title: How to Configure High Availability - Complete Beginner's Guide
---

# How to Configure High Availability - Complete Beginner's Guide

## Question: How do I set up High Availability?

### Answer: Follow these steps

---

## Prerequisites

- 3+ nodes cluster
- Shared storage
- Quorum

---

## How to Enable HA

### Step 1: Add Cluster HA
```bash
# Cluster must exist first
pvecm create mycluster
```

### Step 2: Create HA Group
```bash
ha-manager group add production pve1 pve2 pve3
```

### Step 3: Enable HA for VM
```bash
ha-manager enable vm:100 --group production
```

---

## How to Check HA Status

```bash
ha-manager status
```

---

## Keywords

#ha #high-availability #how-to #beginner #cluster

## Related Articles

- [[High-Availability]]
- [[Cluster]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
