---
title: VPN Configuration
tags:
  - vpn
  - network
  - security
  - wireguard
  - openvpn
---

# VPN Configuration

## Overview

Set up VPN for remote access to Proxmox and VMs.

## WireGuard VPN

### Install

```bash
apt install wireguard -y
```

### Server Config

```bash
# Generate keys
wg genkey | tee /etc/wireguard/serverPrivate.key
wg pubkey < /etc/wireguard/serverPrivate.key > /etc/wireguard/serverPublic.key

# Create config /etc/wireguard/wg0.conf
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <server-private-key>
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT
PostUp = iptables -t nat -A POSTROUTING -o vmbr0 -j MASQUERADE

[Peer]
PublicKey = <client-public-key>
AllowedIPs = 10.0.0.2/32
```

### Client Config

```
[Interface]
Address = 10.0.0.2/24
PrivateKey = <client-private-key>

[Peer]
Endpoint = your-server.com:51820
PublicKey = <server-public-key>
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

## OpenVPN

### Install

```bash
apt install openvpn easy-rsa -y
```

### Create CA

```bash
build-ca
build-key-server server
build-key client
```

### Create Server

```bash
# Create server config
openvpn --genkey --secret /etc/openvpn/server.key
```

## Proxmox VPN for Remote Access

```bash
# Access Proxmox UI remotely
# Configure VPN first, then access via VPN IP
```

## See Also

- [[index|Back to Proxmox VE]]
- [[Security]]
- [[Remote-Access]]