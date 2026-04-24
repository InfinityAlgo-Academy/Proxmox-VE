# Ansible Integration Guide

## Overview

Use Ansible to automate Proxmox VE VM and container management with playbooks and roles.

## Setup

### Install Ansible

```bash
pip install ansible
# or
apt install ansible
```

### Install Proxmox Module

```bash
# Via galaxy
ansible-galaxy collection install community.general

# Or from source
git clone https://github.com/ansible-collections/community.general.git
```

## Configure Connection

### Inventory

```ini
# inventory.ini
[pve]
pve1 ansible_host=pve1.example.com
pve2 ansible_host=pve2.example.com

[pve:vars]
ansible_user=root@pam
ansible_python_interpreter=/usr/bin/python3
```

### Variables

```yaml
# group_vars/pve.yml
ansible_connection: local
pm_api_url: https://pve.example.com:8006/api2/json
pm_api_token_id: root@pam!ansible
pm_api_token_secret: your-token-secret
```

## VM Management

### Create VM

```yaml
# create-vm.yml
- name: Create VM
  hosts: pve1
  gather_facts: no
  tasks:
    - name: Create VM
      community.general.proxmox_vm:
        api_user: root@pam
        api_token_id: ansible
        api_token_secret: "{{ pm_token }}"
        api_host: pve1.example.com
        vmid: 900
        name: my-vm
        node: pve1
        os_type: l26
        memory: 4096
        cores: 2
        disk:
          size: 32G
          storage: local
        network:
          model: virtio
          bridge: vmbr0
        state: present
```

### Start VM

```yaml
# start-vm.yml
- name: Start VM
  hosts: pve1
  tasks:
    - name: Start VM
      community.general.proxmox_vm:
        vmid: 900
        state: started
```

### Stop VM

```yaml
# stop-vm.yml
- name: Stop VM
  hosts: pve1
  tasks:
    - name: Stop VM
      community.general.proxmox_vm:
        vmid: 900
        state: stopped
```

### Delete VM

```yaml
# delete-vm.yml
- name: Delete VM
  hosts: pve1
  tasks:
    - name: Delete VM
      community.general.proxmox_vm:
        vmid: 900
        state: absent
```

## Container Management

### Create Container

```yaml
# create-ct.yml
- name: Create LXC
  hosts: pve1
  tasks:
    - name: Create container
      community.general.proxmox_lxc:
        vmid: 200
        hostname: my-container
        node: pve1
        ostemplate: local:vztmpl/debian-12.tar.gz
        rootfs: local:8
        memory: 2048
        cores: 2
        network:
          name: eth0
          bridge: vmbr0
          ip: dhcp
        state: present
```

### Start Container

```yaml
# start-ct.yml
- name: Start container
  hosts: pve1
  tasks:
    - name: Start LXC
      community.general.proxmox_lxc:
        vmid: 200
        state: started
```

## Cluster Operations

### Get Cluster Status

```yaml
- name: Check cluster
  hosts: pve1
  tasks:
    - name: Get cluster status
      community.general.proxmox_cluster_info:
      register: cluster_info
    
    - name: Display status
      debug:
        var: cluster_info
```

## Backup Automation

### Backup VM

```yaml
# backup-vm.yml
- name: Backup VM
  hosts: pve1
  tasks:
    - name: Create backup
      community.general.proxmox_vzdump:
        vmid: 900
        storage: local
        mode: snapshot
        compress: zstd
```

## User Management

### Create User

```yaml
- name: Manage users
  hosts: pve1
  tasks:
    - name: Create user
      community.general.proxmox_user:
        user: newuser@example.com
        password: "{{ new_password }}"
        state: present
```

### Create Token

```yaml
- name: Create API token
  hosts: pve1
  tasks:
    - name: Create token
      community.general.proxmox_api_token:
        user: root@pam
        tokenid: ansible
        state: present
```

## Complete Playbook

```yaml
# deploy-app.yml
- name: Deploy application infrastructure
  hosts: pve1
  vars:
    app_name: myapp
    app_vms:
      - { vmid: 910, memory: 4096 }
      - { vmid: 911, memory: 4096 }
      - { { vmid: 912, memory: 8192 }
  
  tasks:
    - name: Create app VMs
      community.general.proxmox_vm:
        vmid: "{{ item.vmid }}"
        name: "{{ app_name }}-{{ item.vmid }}"
        node: pve1
        os_type: l26
        memory: "{{ item.memory }}"
        cores: 2
        disk:
          size: "32G"
          storage: local
        network:
          model: virtio
          bridge: vmbr0
        state: present
      loop: "{{ app_vms }}"
    
    - name: Start VMs
      community.general.proxmox_vm:
        vmid: "{{ item.vmid }}"
        state: started
      loop: "{{ app_vms }}"
```

## Keywords

#ansible #automation #playbook #orchestration #devops

## Related Articles

- [[API]]
- [[Scripts]]
- [[Terraform]]
---

[[index|Back to Proxmox VE]]
