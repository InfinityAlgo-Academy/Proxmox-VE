# Automation Scripts Best Practices

## Overview

Best practices for writing automation scripts in Proxmox VE. Learn to create reliable, maintainable, and secure automation scripts.

## Script Structure

### Basic Template

```bash
#!/bin/bash
#
# script-name.sh - Description
# Author: Name
# Date: $(date +%Y-%m-%d)

set -euo pipefail

# Variables
CONFIG_FILE="/root/config"

# Functions
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*"
}

error() {
    echo "[ERROR] $*" >&2
    exit 1
}

# Main
main() {
    log "Starting script..."
    
    # Your code here
    
    log "Complete"
}

main "$@"
```

### Advanced Template

```bash
#!/bin/bash
#
# script-name.sh - Description
set -euo pipefail

VERSION="1.0"
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
LOG_FILE="/var/log/script-name.log"

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

# Logging
log_info() { printf "${GREEN}[INFO]${NC} %s\n" "$@"; }
log_warn() { printf "${YELLOW}[WARN]${NC} %s\n" "$@"; }
log_error() { printf "${RED}[ERROR]${NC} %s\n" "$@" >&2; }

# Cleanup
cleanup() {
    # Cleanup code
    :
}
trap cleanup EXIT

# Main
main() {
    # Check root
    [[ $EUID -eq 0 ]] || { log_error "Must run as root"; exit 1; }
    
    log_info "Script v$VERSION started"
    
    # Your code here
    
    log_info "Complete"
}

main "$@"
```

## Error Handling

### Basic Error Handling

```bash
# Exit on error
set -e

# Exit on undefined variable
set -u

# Combine
set -eu
```

### Advanced Error Handling

```bash
#!/bin/bash
set -euo pipefail

# Function with error handling
run_command() {
    local cmd="$1"
    log "Running: $cmd"
    eval "$cmd" || { error "Command failed: $cmd"; return 1; }
}

# Check result
if run_command "qm start 100"; then
    log "Success"
else
    error "Failed"
fi
```

## Input Validation

### Validate Input

```bash
#!/bin/bash

VMID="${1:-}"
[ -z "$VMID" ] && { echo "Usage: $0 <vmid>"; exit 1; }

# Numeric check
[[ "$VMID" =~ ^[0-9]+$ ]] || { echo "Invalid VMID"; exit 1; }

# Range check
(( VMID >= 100 && VMID <= 999 )) || { echo "VMID out of range"; exit 1; }

# Existence check
qm list | grep -q "^$VMID " || { echo "VM $VMID not found"; exit 1; }
```

### Validate Environment

```bash
#!/bin/bash

# Check network
ping -c 1 -W 2 8.8.8.8 >/dev/null 2>&1 || { echo "No network"; exit 1; }

# Check storage
pvesm status | grep -q local || { echo "Local storage not available"; exit 1; }

# Check disk space
df /var/lib/vz | awk 'NR==2 {exit ($4 < 10)}' || { echo "Low disk space"; exit 1; }
```

## Safety Features

### Lock Files

```bash
LOCKFILE="/var/lock/script.lock"

acquire_lock() {
    if [ -f "$LOCKFILE" ]; then
        # Check if process is still running
        local pid
        pid=$(cat "$LOCKFILE" 2>/dev/null)
        if [ -n "$pid" ] && kill -0 "$pid" 2>/dev/null; then
            echo "Already running (PID: $pid)"
            exit 1
        fi
        rm -f "$LOCKFILE"
    fi
    echo $$ > "$LOCKFILE"
}

release_lock() {
    rm -f "$LOCKFILE"
}

trap release_lock EXIT
acquire_lock
```

### Dry Run Mode

```bash
#!/bin/bash
DRY_RUN=false

while [[ $# -gt 0 ]]; do
    case $1 in
        --dry-run) DRY_RUN=true; shift;;
        *) shift;;
    esac
done

run() {
    if $DRY_RUN; then
        echo "[DRY RUN] Would execute: $*"
    else
        eval "$@"
    fi
}

run qm start 100
```

## Logging

### File Logging

```bash
#!/bin/bash

LOG_FILE="/var/log/script.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}
```

### Syslog Logging

```bash
#!/bin/bash

log() {
    logger -t script-name "$@"
}

log "Starting"
log "Complete"
```

## Comments and Documentation

### Good Comments

```bash
#!/bin/bash
# backup-vms.sh - Backup all running VMs to local storage
# 
# Usage: ./backup-vms.sh [vmids...]
# Example: ./backup-vms.sh 100 101 102

# Exit on any error (set -e) and undefined variable (set -u)
set -eu

# VM list - default to all running VMs if not provided
VM_LIST="${*:$(qm list | grep running | awk '{print $1}' | tr '\n' ' ')}"

# Main backup function
backup_vm() {
    local vmid="$1"
    echo "Backing up VM $vmid..."
    vzdump --mode suspend --vmid "$vmid" --storage local
}
```

## Testing Scripts

### Test Mode

```bash
#!/bin/bash

# Simple test mode
TEST=true

if $TEST; then
    echo "Would execute: $*"
else
    eval "$@"
fi
```

### Validation Script

```bash
#!/bin/bash
# validate-script.sh - Check script for common issues

# Check for set -e
grep -q 'set -e' "$1" || echo "WARNING: No 'set -e'"

# Check for error handling
grep -q 'set -u' "$1" || echo "WARNING: No 'set -u'"

# Check execute permission
[ -x "$1" ] || echo "WARNING: Not executable"

# Run shellcheck if available
command -v shellcheck >/dev/null && shellcheck "$1"
```

## Keywords

#scripting #bash #automation #best-practices #template

## Related Articles

- [[Automation]]
- [[Cron-Jobs]]
- [[Scripts]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
