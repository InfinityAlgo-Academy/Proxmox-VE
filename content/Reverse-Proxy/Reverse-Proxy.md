---
title: Reverse Proxy
tags:
  - reverse-proxy
  - nginx
  - haproxy
  - web
  - proxy
---

# Reverse Proxy

## Overview

Reverse proxy routes traffic to multiple services using a single IP.

## Nginx Reverse Proxy

### Install

```bash
apt install nginx -y
```

### Configuration

```nginx
# /etc/nginx/sites-available/proxmox
server {
    listen 80;
    server_name proxmox.example.com;

    location / {
        proxy_pass https://192.168.1.100:8006;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Multiple Services

```nginx
server {
    listen 80;
    server_name services.example.com;

    location /proxmox/ {
        proxy_pass https://192.168.1.100:8006/;
    }

    location /jellyfin/ {
        proxy_pass http://192.168.1.101:8096/;
    }

    location /nextcloud/ {
        proxy_pass http://192.168.1.102:80/;
    }
}
```

## HAProxy

### Install

```bash
apt install haproxy -y
```

### Configuration

```bash
# /etc/haproxy/haproxy.cfg
frontend http_in
    bind *:80
    bind *:443
    option httpchk

    use_backend proxmox if { path_beg /api2 /json }
    use_backend jellyfin if { path_beg /jellyfin }
    use_backend nextcloud if { path_beg /nextcloud }

backend proxmox
    server pve1 192.168.1.100:8006 check ssl verify none

backend jellyfin
    server jellyfin 192.168.1.101:8096 check

backend nextcloud
    server nextcloud 192.168.1.102:80 check
```

## SSL Termination

```bash
# Generate SSL certificate
apt install certbot -y
certbot -d services.example.com
```

## See Also

- [[index|Back to Proxmox VE]]
- [[Web-Apps]]
- [[Security]]