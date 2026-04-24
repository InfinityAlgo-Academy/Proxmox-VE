# Proxmox VE Integrations Guide

## Table of Contents

1. [External Integrations](#external-integrations)
2. [Proxmox VE API](#proxmox-ve-api)
3. [Webhooks](#webhooks)
4. [Third-Party Tools](#third-party-tools)
5. [Custom Integrations](#custom-integrations)
6. [Automation Tools](#automation-tools)

---

## External Integrations

### Ansible Integration

```bash
# Install Ansible
apt install -y ansible

# Create inventory
mkdir -p /etc/ansible
cat > /etc/ansible/hosts << 'EOF'
[pve]
pve1 ansible_host=192.168.1.100

[nodes]
pve1 ansible_host=192.168.1.100

[vm]
vm1 ansible_host=192.168.1.101
EOF

# Create playbook
cat > /root/pve.yml << 'EOF'
- hosts: pve
  gather_facts: no
  tasks:
    - name: List VMs
      command: qm list
      register: vms
    - debug: var=vms
EOF

# Run playbook
ansible-playbook /root/pve.yml
```

### Terraform Integration

```hcl
# main.tf
provider "proxmox" {
  pm_api_url = "https://192.168.1.100:8006/api2/json/"
  pm_user = "root@pam"
  pm_password = "password"
  pm_insecure = true
}

resource "proxmox_vm_qemu" "vm" {
  name = "terraform-vm"
  target_node = "pve1"
  memory = 2048
  cores = 2
  disk {
    size = 20
    type = "scsi"
    storage = "local"
  }
  network {
    model = "virtio"
    bridge = "vmbr0"
  }
}
```

### Puppet Integration

```bash
# Install Puppet
apt install -y puppet

# Create manifest
cat > /etc/puppet/manifests/nodes.pp << 'EOF'
node 'pve1' {
  package { 'htop':
    ensure => installed,
  }
}
EOF
```

---

## Proxmox VE API

### API Endpoints

```bash
# Using pvesh
pvesh get /cluster/status
pvesh get /nodes
pvesh get /cluster/resources
```

### Custom API Script

```python
import requests

PROXMOX_HOST = "https://192.168.1.100:8006"
USER = "root@pam"
PASSWORD = "password"

def get_token():
    response = requests.post(
        f"{PROXMOX_HOST}/api2/json/access/ticket",
        data={"username": USER, "password": PASSWORD}
    )
    return response.json()["data"]["ticket"]

def get_vms():
    token = get_token()
    headers = {"Cookie": f"PVEAuthCookie={token}"}
    response = requests.get(
        f"{PROXMOX_HOST}/api2/json/cluster/resources",
        headers=headers
    )
    return response.json()["data"]
```

---

## Webhooks

### Configure Webhook

```bash
# Create webhook handler
mkdir -p /root/webhooks

cat > /root/webhooks/vm-webhook.py << 'EOF'
#!/usr/bin/env python3
import requests
import json
import sys

WEBHOOK_URL = "https://discord.com/api/webhooks/xxx"

def send_webhook(message):
    data = {"content": message}
    requests.post(WEBHOOK_URL, json=data)

if __name__ == "__main__":
    event = json.load(sys.stdin)
    send_webhook(f"VM Event: {event}")
EOF

chmod +x /root/webhooks/vm-webhook.py
```

---

## Third-Party Tools

### Zabbix Monitoring

```bash
# Install Zabbix agent
apt install -y zabbix-agent

# Configure
# /etc/zabbix/zabbix_agentd.conf
Server=192.168.1.200
ServerActive=192.168.1.200
Hostname=pve1

systemctl enable zabbix-agent
systemctl start zabbix-agent
```

### Nagios Integration

```bash
# Install NRPE
apt install -y nagios-nrpe-plugin

# Create check script
cat > /usr/lib/nagios/plugins/check_pve << 'EOF'
#!/bin/bash
qm list | grep running | wc -l
EOF

chmod +x /usr/lib/nagios/plugins/check_pve
```

---

## Custom Integrations

### Custom Events

```bash
# Create event handler
cat > /etc/pve/qemu-server/<vmid>.conf << 'EOF'
hook: /root/scripts/vm-hook.sh
EOF

cat > /root/scripts/vm-hook.sh << 'EOF'
#!/bin/bash
ACTION=$1
VMID=$2

case $ACTION in
    start)
        echo "VM $VMID started" >> /var/log/pve-events.log
        ;;
    stop)
        echo "VM $VMID stopped" >> /var/log/pve-events.log
        ;;
esac
EOF
```

---

## Automation Tools

### Jenkins Integration

```groovy
// Jenkinsfile
pipeline {
    agent any
    stages {
        stage('Create VM') {
            steps {
                sh 'qm create 100 --name jenkins-vm --memory 4096'
            }
        }
        stage('Configure VM') {
            steps {
                sh 'qm set 100 --cores 4'
            }
        }
    }
}
```

### GitLab CI Integration

```yaml
# .gitlab-ci.yml
create-vm:
  script:
    - qm create $VM_ID --name $VM_NAME --memory 4096
  only:
    - main

destroy-vm:
  script:
    - qm stop $VM_ID
    - qm destroy $VM_ID
  when: manual
```

---

## Keywords

#integration #ansible #terraform #api #webhooks #automation

## Links

- [[Proxmox-VE]] - Main documentation
- [[API]] - API documentation
- [[Scripts]] - Automation scripts
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
