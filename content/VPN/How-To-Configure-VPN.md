# How to Set Up VPN - Complete Beginner's Guide

## Question: How do I set up VPN access?

### Answer: Multiple options available

---

## Option 1: WireGuard (Recommended)

### Step 1: Install
```bash
apt install wireguard
```

### Step 2: Configure
```bash
# Generate keys
wg genkey | tee private.key
wg pubkey < private.key

# Edit config
nano /etc/wireguard/wg0.conf
```

---

## Option 2: OpenVPN

### Install
```bash
apt install openvpn easy-rsa
```

---

## Keywords

#vpn #how-to #beginner #wireguard #openvpn

## Related Articles

- [[VPN]]
- [[Remote-Access]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
