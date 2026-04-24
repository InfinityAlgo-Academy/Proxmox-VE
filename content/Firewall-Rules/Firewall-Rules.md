---
title: Firewall Rules
---

# Firewall Rules

## Proxmox Firewall

```bash
# Enable firewall
pvenode modify --firewall 1

# Add rule
iptables -A INPUT -p tcp --dport 8006 -j ACCEPT

# List rules
iptables -L -n
```

## VM Firewall

```bash
# Enable VM firewall
qm set 100 --net0 virtio,firewall=1

# Add rule
pvesh create /nodes/localhost/firewall/rules --enable 1 --rule "$(cat <<EOF
{
  "action": "ACCEPT",
  "protocol": "tcp",
  "dport": 80,
  "comment": "Allow HTTP"
}
EOF
)"
```

## See Also

- [[index|Back to Proxmox VE]]