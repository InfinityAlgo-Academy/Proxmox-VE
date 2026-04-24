# How to Enable Beta Features - Complete Guide

## Question: How do I enable and test beta features in Proxmox?

### Answer: Enable PVE beta repository

---

## How to Enable Beta Repository

### Step 1: Add Testing Repo
```bash
# Add PVE beta repository
echo "deb http://download.proxmox.com/debian/pve bookworm pve-test" > \
  /etc/apt/sources.list.d/pve-beta.list

apt update
```

---

## How to Install Beta Packages

### Step 1: Install Specific Beta
```bash
# Install specific beta version
apt install proxmox-ve=8.0-1~beta1

# Or upgrade to beta
apt full-upgrade -y --allow-remove-essential
```

---

## How to Revert to Stable

### Step 1: Downgrade
```bash
# Remove beta repo
rm /etc/apt/sources.list.d/pve-beta.list

# Downgrade
apt update
apt install proxmox-ve/stable
```

---

## Keywords

#beta-features #testing #how-to #preview

## Related Articles

- [[Beta-Features]]
- [[Proxmox-VE]]
- [[Updates]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
