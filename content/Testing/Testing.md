---
title: Testing
---

# Testing Proxmox

## Test Network

```bash
# Test connectivity
ping -c 4 8.8.8.8

# Test DNS
nslookup proxmox.local

# Test speed
iperf3 -s &
iperf3 -c 192.168.1.1
```

## Test Storage

```bash
# Test disk speed
dd if=/dev/zero of=/tmp/test bs=1M count=1000
hdparm -t /dev/sda
```

## Test VM

```bash
# Test VM performance
qm monitor 100
info balloon
info block
```

## See Also

- [[index|Back to Proxmox VE]]