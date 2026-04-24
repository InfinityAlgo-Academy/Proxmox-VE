# How to Install Docker in Proxmox VE - Complete Beginner's Guide

## Question: How do I install Docker in Proxmox?

### Answer: Follow these simple steps

---

## Option 1: Install Docker in LXC Container (Recommended)

### Step 1: Create Container
1. Create new LXC container (Ubuntu or Debian)
2. Use at least 2 CPU, 4GB RAM

### Step 2: Install Docker
```bash
apt update && apt install -y docker.io
systemctl enable docker
systemctl start docker
```

### Step 3: Test Docker
```bash
docker run hello-world
```

---

## Option 2: Install Docker in VM

### Steps:
1. Create new VM with Ubuntu
2. Install Docker:
```bash
curl get.docker.com | sh
```

---

## How to Install Portainer for Web UI

```bash
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer
```

Access at: http://your-ip:9000

---

## Keywords

#docker #how-to #beginner #container #portainer

## Related Articles

- [[Docker]]
- [[Proxmox-VE]]
- [[Containers]]
---

[[index|Back to Proxmox VE]]
