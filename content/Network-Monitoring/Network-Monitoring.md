---
title: Network Monitoring
---

# Network Monitoring

## Monitor Network

```bash
# Check network stats
pvesh get /nodes/localhost/network

# Show interfaces
ip addr
ip link

# Check traffic
iftop
nethogs
```

## Monitoring Tools

| Tool | Description |
|-----|-------------|
| `iftop` | Monitor bandwidth |
| `nethogs` | Per-process traffic |
| `tcpdump` | Packet capture |
| `iperf3` | Speed test |

## See Also

- [[index|Back to Proxmox VE]]