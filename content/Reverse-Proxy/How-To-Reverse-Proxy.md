---
title: How to Set Up Reverse Proxy - Complete Beginner's Guide
---

# How to Set Up Reverse Proxy - Complete Beginner's Guide

## Question: How do I set up a reverse proxy?

### Answer: Using Nginx or Traefik

---

## Option 1: Nginx Reverse Proxy

### Step 1: Install
```bash
apt install nginx
```

### Step 2: Configure
```bash
# Create config
nano /etc/nginx/sites-available/myapp

# Add:
server {
    listen 80;
    server_name app.yourdomain.com;
    location / {
        proxy_pass http://192.168.1.100:80;
    }
}
```

### Step 3: Enable
```bash
ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

---

## Option 2: Traefik

```bash
# Run as container
docker run -d -p 80:80 -p 8080:8080 traefik
```

---

## Keywords

#nginx #reverseproxy #how-to #beginner #traefik

## Related Articles

- [[Reverse-Proxy]]
- [[Web-Apps]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]
