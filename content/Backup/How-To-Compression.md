# How to Optimize Backup Compression - Complete Guide

## Question: How do I compress backups to save space?

### Answer: Use built-in compression options

---

## Understanding Compression Options

| Compression | Speed | Space Saved | CPU Use |
|-------------|-------|------------|---------|
| **none** | Fastest | None | Low |
| **lzo** | Fast | ~40% | Low |
| **gzip** | Medium | ~50% | Medium |
| **zstd** | Fast | ~55% | Medium |

---

## How to Enable Compression

### Method 1: Web UI

**Step 1: Edit Backup Job**
- Go to **Datacenter** → **Backup**
- Click on backup job → **Edit**

**Step 2: Select Compression**
Find **Compression** dropdown:
- Select **lzo** (fast) or **zstd** (better compression)

**Step 3: Save**
- Click **OK**

---

### Method 2: CLI

```bash
# Use LZO compression
vzdump --mode snapshot --vmid 100 --compress lzo

# Use ZSTD compression (recommended)
vzdump --mode snapshot --vmid 100 --compress zstd

# Use gzip
vzdump --mode snapshot --vmid 100 --compress gzip

# No compression
vzdump --mode snapshot --vmid 100 --compress none
```

---

## How to Check Compression Results

```bash
# List backups with sizes
ls -lh /var/lib/vz/dump/

# Compare file sizes
du -h /var/lib/vz/dump/*

# Check compression ratio
gzip -l /var/lib/vz/dump/vzdump-qemu-100-2024.tar.gz
```

---

## Pro Tips

1. **Use zstd** for best compression
2. **Use lzo** for faster backups
3. **Test both** and compare results

---

## Keywords

#compression #zstd #lzo #how-to #optimization

## Related Articles

- [[Backup]]
- [[How-To-Configure-Backup]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
