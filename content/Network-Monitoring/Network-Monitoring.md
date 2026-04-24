---
title: Network Monitoring
tags:
  - network
  - monitoring
  - troubleshooting
  - bandwidth
---

# Network Monitoring

## Monitor Network Interfaces

```bash
# Show interfaces
ip addr
ip link

# Show statistics
ip -s link
```

## Bandwidth Monitoring

### iftop - Real-time traffic

```bash
# Install
apt install iftop -y

# Monitor
iftop

# Specific interface
iftop -i vmbr0
```

### nethogs - Per-process traffic

```bash
# Install
apt install nethogs -y

# Monitor
nethogs

# Monitor specific interface
nethogs eth0
```

## Packet Capture

### tcpdump

```bash
# Capture all packets
tcpdump -i eth0

# Capture specific port
tcpdump -i eth0 port 80

# Write to file
tcpdump -i eth0 -w capture.pcap
```

### tshark

```bash
# Install
apt install wireshark-cli -y

# Capture
tshark -i eth0 -c 100
```

## Network Statistics

```bash
# Show connections
ss -s

# Show socket statistics
ss -tuln

# Network interfaces
netstat -i

# Routing table
netstat -rn
```

## Grafana Monitoring

- See [[Monitoring-Tools]]

## See Also

- [[index|Back to Proxmox VE]]
- [[Logging]]
- [[Performance]]