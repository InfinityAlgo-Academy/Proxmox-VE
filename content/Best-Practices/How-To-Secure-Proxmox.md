# How to Secure Proxmox VE After Installation - Complete Guide

## Question: How do I secure my Proxmox server after install?

### Answer: Follow these essential security steps

---

## How to Update System Immediately

### Step 1: Add Community Repository
```bash
# Disable enterprise repo
sed -i 's/^deb/#deb/' /etc/apt/sources.list.d/pve-enterprise.list

# Add community repo
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-community.list
```

### Step 2: Update
```bash
apt update && apt full-upgrade -y
reboot
```

---

## How to Secure SSH Access

### Step 1: Disable Root Login
```bash
sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
systemctl restart sshd
```

### Step 2: Use SSH Keys
```bash
# Generate key on your computer
ssh-keygen -t ed25519

# Copy to server
ssh-copy-id admin@your-server-ip
```

---

## How to Enable Firewall

### Step 1: Install UFW
```bash
apt install ufw -y
```

### Step 2: Configure Rules
```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow 8006/tcp   # Proxmox Web
ufw allow 22/tcp      # SSH

# Enable
ufw enable
```

---

## How to Enable 2FA

### Step-by-Step (Web UI)
1. Go to **Datacenter** → **Users**
2. Select user → **2FA** tab
3. Click **Add** → **TOTP**
4. Scan with authenticator app
5. Enter code to enable

---

## Keywords

#security #ssh #firewall #2fa #how-to #hardening

## Related Articles

- [[Best-Practices]]
- [[Security]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
