---
title: Access Auditing and Compliance
---

# Access Auditing and Compliance

## Overview

Access auditing tracks who accessed what resources and when. Essential for security compliance, incident investigation, and regulatory requirements.

## Audit Logs

### Viewing Access Logs

#### Via Web UI

1. Navigate to **Datacenter → Logs → Access**
2. Filter by:
   - Date range
   - User
   - Resource
   - Action type

#### Via CLI

```bash
# View all access logs
pveam access log

# Filter by user
pveam access log --user admin@example.com

# Filter by date
pveam access log --start 2025-01-01 --end 2025-01-31

# Filter by action
pveam access log --user admin@example.com --action power
```

### Log Contents

Each log entry includes:
- **Timestamp**: When action occurred
- **User**: Who performed action
- **IP Address**: Source of request
- **Action**: What was done
- **Resource**: Target of action
- **Result**: Success/failure

## User Activity Tracking

### Login Tracking

```bash
# View login history
pveam access log --action login

# View failed logins
pveam access log --action login --failed 1

# View concurrent sessions
pveam access sessions
```

### Action Tracking

```bash
# Track VM operations
pveam access log --action "vm:start"
pveam access log --action "vm:stop"
pveam access log --action "vm:create"

# Track configuration changes
pveam access log --action config
pveam access log --action modify
```

## Compliance Reports

### User Access Report

```bash
# Generate user access report
pveum user report --output /tmp/user-access-report.csv

# Report includes:
# - User list
# - Last login
# - Active sessions
# - Permissions
# - 2FA status
```

### Permission Audit Report

```bash
# Generate permission report
pveum acl report --output /tmp/permission-report.csv

# Report includes:
# - All ACLs
# - User/group assignments
# - Role assignments
# - Path coverage
```

### Quarterly Audit Template

Quarterly access review checklist:
1. User list verification
2. Permission review
3. 2FA compliance check
4. Expired account check
5. Orphaned permission check

## Anomaly Detection

### Failed Login Alerts

```bash
# View repeated failures
pveam access log --failed 1 --count 5

# Configure alerts
# Datacenter → Logs → Alerts → Add
# Type: Failed Login
# Threshold: 3 attempts
# Window: 15 minutes
```

### Unusual Access Patterns

```bash
# Detect off-hours access
pveam access log --start 18:00 --end 08:00

# Detect resource spam
pveam access log --action vm:create --count 10 --hour 1
```

## Integration with SIEM

### Syslog Export

```bash
# Configure syslog
pveam access syslog --enable --server syslog.example.com --port 514

# Log format: CEF (Common Event Format)
```

### Webhook Notifications

```bash
# Configure webhook
pveam access notify --webhook https://hooks.example.com/siem

# Send: all access events
```

## Compliance Requirements

### HIPAA Compliance

For healthcare environments:
- Track all PHI access
- Maintain 6-year audit logs
- Enable 2FA for all users
- Regular access reviews

### PCI DSS Compliance

For payment environments:
- Unique user IDs
- Access logging
- 2FA for remote access
- Quarterly access reviews

### SOX Compliance

For financial environments:
- Track all configuration changes
- Log all data access
- Maintain 7-year audit logs
- Regular access certification

## Best Practices

1. **Enable comprehensive logging**
2. **Regular log review** (weekly minimum)
3. **Automated alerts** for anomalies
4. **Export for analysis** (monthly)
5. **Document exceptions**

## Related Articles

- [[Roles and Permissions]]
- [[Two-Factor Authentication]]
- [[Access Control Lists]]
- [[Security Best Practices]]
---

[[index|Back to Proxmox VE]]
