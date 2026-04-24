---
title: Database
---

# Database on Proxmox

## MariaDB/MySQL

```bash
# Install MariaDB
apt install mariadb-server -y

# Start service
systemctl start mariadb
systemctl enable mariadb
```

## PostgreSQL

```bash
# Install PostgreSQL
apt install postgresql -y

# Start service
systemctl start postgresql
systemctl enable postgresql
```

## Run in Container

```bash
# Create database container
pct create 200 \
  --storage local \
  --rootfs local:10 \
  --ostype debian \
  --memory 2048
```

## See Also

- [[Containers]]
- [[Docker]]
- [[VM]]
[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
