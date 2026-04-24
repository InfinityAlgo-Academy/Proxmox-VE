# Proxmox VE Network Services Guide

## Table of Contents

1. [DNS Services](#dns-services)
2. [DHCP Services](#dhcp-services)
3. [Proxy Services](#proxy-services)
4. [VPN Services](#vpn-services)
5. [Firewall Services](#firewall-services)

---

## DNS Services

### dnsmasq

```bash
# Install
apt install -y dnsmasq

# Configure /etc/dnsmasq.conf
interface=vmbr0
bind-interfaces
dhcp-range=192.168.1.200,192.168.1.250,12h
dhcp-option=option:router,192.168.1.1

# Start
systemctl enable dnsmasq
systemctl start dnsmasq
```

### Unbound

```bash
# Install
apt install -y unbound

# Configure /etc/unbound/unbound.conf
server:
    interface: 0.0.0.0
    access-control: 192.168.1.0/24 allow

# Start
systemctl enable unbound
systemctl start unbound
```

---

## DHCP Services

### ISC DHCP

```bash
# Install
apt install -y isc-dhcp-server

# Configure /etc/dhcp/dhcpd.conf
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.200 192.168.1.250;
    option routers 192.168.1.1;
}

# Start
systemctl enable isc-dhcp-server
systemctl start isc-dhcp-server
```

---

## Proxy Services

### Squid

```bash
# Install
apt install -y squid

# Configure /etc/squid/squid.conf
http_port 3128
acl localnet src 192.168.1.0/24
http_access allow localnet

# Start
systemctl enable squid
systemctl start squid
```

### Nginx

```bash
# Install
apt install -y nginx

# Configure
server {
    listen 80;
    server_name myapp.local;
    location / {
        proxy_pass http://192.168.1.100:8080;
    }
}

# Start
systemctl enable nginx
systemctl start nginx
```

---

## VPN Services

### WireGuard

```bash
# Install
apt install -y wireguard wireguard-tools

# Create keys
wg genkey | tee privatekey | wg pubkey > publickey

# Configure /etc/wireguard/wg0.conf
[Interface]
Address = 10.0.0.1/24
PrivateKey = <private-key>
ListenPort = 51820

[Peer]
PublicKey = <peer-public-key>
AllowedIPs = 10.0.0.2/32

# Start
wg-quick up wg0
systemctl enable wg-quick@wg0
```

### OpenVPN

```bash
# Install
apt install -y openvpn easy-rsa

# Setup
make-cadir /etc/openvpn/easy-rsa
cd /etc/openvpn/easy-rsa
./easyrsa build-server-full server nopass

# Configure server
cp /etc/openvpn/easy-rsa/pki/issued/server.crt /etc/openvpn/server.crt
cp /etc/openvpn/easy-rsa/pki/private/server.key /etc/openvpn/server.key
cp /etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/ca.crt

# Start
systemctl enable openvpn@server
systemctl start openvpn@server
```

---

## Firewall Services

### UFW

```bash
# Install
apt install -y ufw

# Configure
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp

# Enable
ufw enable
```

---

## Keywords

#dns #dhcp #proxy #vpn #firewall

## Links

- [[Proxmox-VE]] - Main documentation
- [[Networking]] - Network configuration
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
