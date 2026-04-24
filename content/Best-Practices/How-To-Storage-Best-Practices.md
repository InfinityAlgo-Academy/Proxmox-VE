# How to Configure Storage Best Practices - Complete Guide

## Question: How do I configure storage properly?

### Answer: Use right storage for right purpose

---

## How to Choose Storage Type

### Storage Selection Guide
| Data Type | Recommended Storage |
|----------|-------------------|
| Boot disk | ZFS Mirror |
| VM disks | LVM-Thin or ZFS |
| Backups | NFS |
| ISO/Template | Directory |
| Fast cache | NVMe |

---

## How to Configure ZFS

### Step 1: Create ZFS Pool
```bash
# Mirror for redundancy
zpool create -f mirror pool1 /dev/sda /dev/sdb
```

### Step 2: Set Properties
```bash
zfs set compression=lz4 pool1
zfs set atime=off pool1
zfs set checksum=sha256 pool1
```

---

## How to Configure LVM-Thin

### Step 1: Create Thin Pool
```bash
lvcreate -L 500G -T vg0/vm-thin
```

---

## How to Configure NFS Storage

### Step 1: Mount NFS
```bash
mount -t nfs -o hard,intr,rsize=8192,wsize=8192 \
  192.168.1.50:/data /mnt/data

# Add to fstab
echo "192.168.1.50:/data /mnt/data nfs hard,intr,rsize=8192,wsize=8192 0 0" >> /etc/fstab
```

---

## Keywords

#storage #zfs #lvm #nfs #how-to

## Related Articles

- [[Best-Practices]]
- [[Storage]]
- [[ZFS-Tuning]]
- [[Proxmox-VE]]