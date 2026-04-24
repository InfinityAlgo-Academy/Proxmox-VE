# Terraform Provider Guide

## Overview

Use Terraform to manage Proxmox VE infrastructure as code. Define VMs, containers, and storage in configuration files.

## Setup

### Install Terraform

```bash
# macOS
brew install terraform

# Linux
apt install terraform

# Download
wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip
unzip terraform_1.6.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/
```

### Install Proxmox Provider

```bash
mkdir -p ~/.terraform.d/plugins
# Provider will be downloaded automatically
```

## Provider Configuration

### Configure Provider

```hcl
# providers.tf

terraform {
  required_providers {
    proxmox = {
      source = "Telmate/proxmox"
      version = "3.0.1"
    }
  }
}

provider "proxmox" {
  pm_api_url = "https://pve.example.com:8006/api2/json"
  pm_api_token_id = "root@pam!terraform"
  pm_api_token_secret = "your-token-secret"
  pm_parallel = 4
  # Or use basic auth
  # pm_user = "root@pam"
  # pm_password = "password"
}
```

## VM Configuration

### Create VM

```hcl
# vm.tf

resource "proxmox_vm_qemu" "web_server" {
  name        = "web-server"
  target_node = "pve1"
  vmid        = 900
  
  iso      = "local:iso/ubuntu-22.04.iso"
  os_type   = "l26"
  cores     = 2
  memory    = 4096
  disk {
    size     = "32G"
    storage = "local"
    type    = "scsi"
  }
  network {
    model   = "virtio"
    bridge  = "vmbr0"
  }
}
```

### VM with Cloud-Init

```hcl
resource "proxmox_vm_qemu" "cloud_server" {
  name        = "cloud-server"
  target_node = "pve1"
  vmid        = 901
  
  os_type   = "l26"
  cores     = 4
  memory    = 8192
  
  disk {
    size     = "50G"
    storage = "local"
    type    = "scsi"
  }
  
  cloudinit {
    storage = "local"
    ipconfig {
      ip = "dhcp"
    }
  }
  
  # SSH key
  sshkeys = file("~/.ssh/id_rsa.pub")
}
```

### Clone VM

```hcl
resource "proxmox_vm_qemu" "cloned_vm" {
  name        = "cloned-vm"
  target_node = "pve1"
  vmid        = 902
  
  clone     = "template-vm"
  clonesnap = "template-snap"
  
  cores     = 2
  memory    = 4096
}
```

## Container Configuration

### Create LXC

```hcl
resource "proxmox_lxc" "app_container" {
  name        = "app-container"
  target_node = "pve1"
  vmid        = 200
  
  ostemplate = "local:vztmpl/debian-12.tar.gz"
  rootfs     = "local:8"
  
  memory    = 2048
  cores     = 2
  
  network {
    name   = "eth0"
    bridge = "vmbr0"
    ip     = "dhcp"
  }
  
  password = "changeme"
  unprivileged = true
}
```

## Storage Configuration

### Define Storage

```hcl
resource "proxmox_storage" "nfs_backup" {
  storage   = "nfs-backup"
  type     = "nfs"
  server   = "192.168.1.50"
  export   = "/backups"
  content  = "vztmpl,iso,backup"
  enable   = true
}
```

## Multiple VMs

### VM Pool

```hcl
# variables.tf
variable "web_vms" {
  type    = list(object({
    name   = string
    vmid  = number
    memory = number
  }))
  default = [
    { name = "web-1", vmid = 910, memory = 4096 },
    { name = "web-2", vmid = 911, memory = 4096 },
    { name = "web-3", vmid = 912, memory = 4096 },
  ]
}

# vm-pool.tf
resource "proxmox_vm_qemu" "web_pool" {
  count    = length(var.web_vms)
  name    = var.web_vms[count.index].name
  target_node = "pve1"
  vmid    = var.web_vms[count.index].vmid
  
  os_type   = "l26"
  cores     = 2
  memory    = var.web_vms[count.index].memory
  disk {
    size     = "32G"
    storage = "local"
  }
  network {
    model   = "virtio"
    bridge  = "vmbr0"
  }
}
```

## Manage Infrastructure

### Initialize

```bash
terraform init
```

### Plan Changes

```bash
terraform plan -out=tfplan
```

### Apply

```bash
terraform apply
```

### Destroy

```bash
terraform destroy
```

## State Management

### Remote Backend

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket = "terraform-state"
    key    = "proxmox/state"
    region = "us-east-1"
  }
}
```

## Keywords

#terraform #infrastructure-as-code #IaC #provisioning #automation

## Related Articles

- [[API]]
- [[API-Authentication]]
- [[Scripts]]