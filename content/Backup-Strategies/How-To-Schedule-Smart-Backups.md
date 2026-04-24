# How to Schedule Smart Backups - Complete Guide

## Question: When should I schedule backups?

### Answer: Schedule during off-peak hours

---

## Understanding Backup Windows

### Off-Peak Hours (Best)
- 1:00 AM - 5:00 AM (typical)
- Weekends
- Holidays

### Peak Hours (Avoid)
- 9:00 AM - 6:00 PM (business hours)
- Monday mornings

---

## How to Schedule (Web UI)

### Step 1: Create Job
1. **Datacenter** → **Backup** → **Add**

### Step 2: Set Schedule
- **Daily**: For critical VMs
- **Weekly**: For full backups
- **Monthly**: For archives

### Step 3: Set Time
Best time: 2:00 AM (off-peak)

---

## How to Schedule (CLI/Cron)

### Daily at 2 AM
```bash
crontab -e
0 2 * * * vzdump --mode snapshot --vmid all --storage local
```

### Multiple Times Per Day
```bash
# 2 AM and 2 PM
0 2,14 * * * vzdump --mode snapshot --vmid all --storage local
```

---

## Smart Scheduling Examples

### Critical VMs (Every 6 Hours)
```bash
0 */6 * * * vzdump --mode snapshot --vmid 100,101,102 --storage local
```

### Non-Critical VMs (Daily)
```bash
0 3 * * * vzdump --mode suspend --vmid all --storage local
```

### Weekly Full + Daily Incrementals
```bash
# Sunday: Full
0 3 * * 0 vzdump --mode suspend --vmid all --storage local

# Mon-Sat: Incremental via PBS
0 3 * * 1-6 vzdump --mode snapshot --vmid all --storage pbs
```

---

## Keywords

#schedule #backup-timing #how-to #cron #automation

## Related Articles

- [[Backup]]
- [[How-To-Schedule-Backup]]
- [[Proxmox-VE]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
