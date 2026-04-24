# How to Perform Regular Maintenance - Complete Guide

## Question: How do I maintain Proxmox regularly?

### Answer: Follow maintenance schedule

---

## Maintenance Schedule

| Task | Frequency |
|------|-----------|
| System update | Weekly |
| Health check | Daily |
| Backup verify | Weekly |
| Storage check | Daily |
| Reboot | Monthly |

---

## How to Update System

### Step 1: Update Weekly
```bash
apt update && apt upgrade -y
reboot
```

### Step 2: Update Containers
```bash
for ct in $(pct list | awk 'NR>1 {print $1}'); do
    pct exec $ct -- bash -c "apt update && apt upgrade -y"
done
```

---

## How to Check Health

### Health Check Script
```bash
#!/bin/bash
# health-check.sh

echo "=== System Load ==="
uptime

echo "=== Services ==="
for svc in pveproxy pvedaemon corosync; do
    systemctl is-active $svc
done

echo "=== Storage ==="
pvesm status

echo "=== VMs ==="
qm list

echo "=== Containers ==="
pct list
```

---

## How to Set Up Log Rotation

### Step 1: Configure
```bash
cat > /etc/logrotate.d/pve << EOF
/var/log/pve*.log {
    daily
    rotate 7
    compress
    delaycompress
    notifempty
}
EOF
```

---

## Keywords

#maintenance #updates #health #how-to #schedule

## Related Articles

- [[Best-Practices]]
- [[Troubleshooting]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
