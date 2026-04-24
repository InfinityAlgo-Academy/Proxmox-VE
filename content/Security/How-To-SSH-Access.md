# How to Configure SSH Access - Complete Beginner's Guide

## Question: How do I access my Proxmox server via SSH?

### Answer: Multiple options available

---

## How to Use SSH

### Enable SSH
```bash
# Install
apt install openssh-server

# Enable
systemctl enable ssh

# Start
systemctl start ssh
```

### Connect from Client
```bash
ssh root@192.168.1.100
```

### Use SSH Key
```bash
ssh -i ~/.ssh/key root@192.168.1.100
```

---

## How to Secure SSH

```bash
# Disable root login
nano /etc/ssh/sshd_config
# PermitRootLogin no

# Restart
systemctl restart ssh
```

---

## Keywords

#ssh #how-to #beginner #remote

## Related Articles

- [[Remote-Access]]
- [[Security]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
