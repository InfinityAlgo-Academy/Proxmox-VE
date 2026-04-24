# Proxmox VE Migration Guide

## Table of Contents

1. [VM Migration](#vm-migration)
2. [Container Migration](#container-migration)
3. [Storage Migration](#storage-migration)
4. [Cluster Migration](#cluster-migration)
5. [Upgrading Migration](#upgrading-migration)

---

## VM Migration

### Online Migration

```bash
# Live migrate VM
qm migrate <vmid> <target-node> --online

# With bandwidth limit
qm migrate <vmid> <target-node> --online --bandwidth 100M

# Migrate multiple VMs
for vm in 100 101 102; do
    qm migrate $vm target-node ---online
done
```

### Offline Migration

```bash
# Offline migrate
qm migrate <vmid> <target-node>

# With storage
qm migrate <vmid> <target-node> --storage target-storage
```

---

## Container Migration

### Online Container Migration

```bash
# Live migrate container
pct migrate <ctid> <target-node> --online

# Migrate with storage
pct migrate <ctid> <target-node> --online --storage local2
```

### Offline Migration

```bash
# Offline migrate
pct migrate <ctid> <target-node>
```

---

## Storage Migration

### Move VM Disk

```bash
# Move disk to new storage
pvesm move local:vm-100-disk-0 local2
```

### Export/Import

```bash
# Export to file
qm export <vmid> qcow2 --storage backup --format qcow2

# Import
qm importdisk <vmid> /path/to/image.qcow2 local
```

---

## Cluster Migration

### Node Migration

```bash
# Leave cluster
pvecm leave

# Add to new cluster
pvecm add <new-cluster-ip>
```

---

## Upgrading Migration

### From Proxmox 7 to 8

```bash
# Backup all VMs
vzdump all --mode stop

# Upgrade
apt update
apt upgrade -y
reboot

# Restore VMs
vzdump restore
```

---

## Keywords

#migration #move #upgrade #export #import

## Links

- [[Proxmox-VE]] - Main documentation
- [[VM]] - VM management
- [[Cluster]] - Clustering
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
