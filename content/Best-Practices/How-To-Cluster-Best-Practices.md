# How to Configure Cluster Best Practices - Complete Guide

## Question: How do I set up a proper cluster?

### Answer: Follow cluster best practices

---

## How to Set Up Cluster

### Prerequisites
- Odd number of nodes (min 3)
- Dedicated network for cluster
- Shared storage

### Step 1: Initial Cluster
```bash
# First node
pvecm create mycluster

# Add second node
pvecm add 192.168.1.101

# Add third node
pvecm add 192.168.1.102
```

---

## How to Configure Quorum

### Step 1: Ensure Quorum
```bash
# Check quorum
pvecm status

# Expected votes should be 3 for 3-node cluster
```

---

## How to Configure HA

### Step 1: Enable HA
```bash
ha-manager group add production pve1 pve2 pve3
ha-manager enable vm:100 --group production
```

---

## Keywords

#cluster #ha #quorum #how-to #high-availability

## Related Articles

- [[Best-Practices]]
- [[Cluster]]
- [[High-Availability]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
