# Go API Client Guide

## Overview

Learn how to use Go to interact with Proxmox VE API for high-performance automation.

## Setup

### Install Library

```bash
go get github.com/lcab/proxmox-go
```

### Using Go Modules

```bash
go mod init proxmox-api
go get github.com/lcab/proxmox-go
```

## Basic Usage

### Connect to API

```go
package main

import (
    "fmt"
    "os"
    
    "github.com/lcab/proxmox-go"
)

func main() {
    client := proxmox.NewClient(
        "https://pve.example.com:8006/api2/json",
        proxmox.TokenCreds("root@pam", "your-token"),
    )
    
    // Get cluster status
    status, err := client.ClusterStatus()
    if err != nil {
        fmt.Println("Error:", err)
        os.Exit(1)
    }
    
    for _, node := range status {
        fmt.Printf("%s: %s\n", node.Node, node.Status)
    }
}
```

## VM Operations

### List VMs

```go
// Get all VMs
resources, err := client.ClusterResources()
if err != nil {
    log.Fatal(err)
}

for _, r := range resources {
    if r.Type == "qemu" {
        fmt.Printf("VM %d: %s (%s)\n", r.VMID, r.Name, r.Status)
    }
}
```

### Create VM

```go
// Create VM
err := client.CreateVM("pve1", proxmox.QEMUConfig{
    VMID:   900,
    Name:   "my-vm",
    Memory: 4096,
    Cores:  2,
    OSType: "l26",
    SCSIDisk: proxmox.ScsiDisk{
        Storage: "local",
        Size:   "32G",
    },
    Net0: proxmox.VirtIO{
        Bridge: "vmbr0",
    },
})
if err != nil {
    log.Fatal(err)
}
```

### Start VM

```go
// Start VM
err := client.StartVM("pve1", 900)
if err != nil {
    log.Fatal(err)
}
```

### Stop VM

```go
// Stop VM
err := client.StopVM("pve1", 900)
if err != nil {
    log.Fatal(err)
}
```

### Get VM Status

```go
// Get VM status
status, err := client.VMStatus("pve1", 900)
if err != nil {
    log.Fatal(err)
}
fmt.Println(status)
```

### Clone VM

```go
// Clone VM
err := client.CloneVM("pve1", 900, 901, "cloned-vm", true)
if err != nil {
    log.Fatal(err)
}
```

### Update VM

```go
// Update VM config
err := client.UpdateVM("pve1", 900, proxmox.QEMUConfig{
    Memory: 8192,
    Cores:  4,
})
if err != nil {
    log.Fatal(err)
}
```

## Container Operations

### Create Container

```go
// Create LXC container
err := client.CreateContainer("pve1", proxmox.LXCConfig{
    VMID:     200,
    OSType:   "debian",
    Hostname: "my-container",
    RootFS: proxmox.RootFS{
        Storage: "local",
        Size:    "8G",
    },
    Memory: 2048,
    Cores:  2,
    Net0: proxmox.LXCNet{
        Name:    "eth0",
        Bridge: "vmbr0",
        IP:      "dhcp",
    },
})
if err != nil {
    log.Fatal(err)
}
```

### Start Container

```go
// Start container
err := client.StartContainer("pve1", 200)
if err != nil {
    log.Fatal(err)
}
```

## Storage Operations

### List Storage

```go
// Get storage
storage, err := client.Storage()
if err != nil {
    log.Fatal(err)
}

for _, s := range storage {
    fmt.Printf("%s: %s (%s/%s)\n", s.Storage, s.Type, s.Avail, s.Total)
}
```

### Get Storage Content

```go
// List content
content, err := client.StorageContent("local")
if err != nil {
    log.Fatal(err)
}

for _, c := range content {
    fmt.Printf("%s: %s\n", c.Volid, c.Size)
}
```

## User Management

### List Users

```go
// Get users
users, err := client.Users()
if err != nil {
    log.Fatal(err)
}

for _, u := range users {
    fmt.Println(u.UserID)
}
```

### Create User

```go
// Create user
err := client.CreateUser("newuser@example.com", "password123")
if err != nil {
    log.Fatal(err)
}
```

### Create Token

```go
// Create API token
token, err := client.CreateUserToken("root@pam", "script-token")
if err != nil {
    log.Fatal(err)
}
fmt.Println(token)
```

## Access Control

### List ACLs

```go
// Get ACLs
acls, err := client.ACLs()
if err != nil {
    log.Fatal(err)
}

for _, a := range acls {
    fmt.Printf("%s: %s -> %s\n", a.Path, a.UserID, a.RoleID)
}
```

### Add ACL

```go
// Add permission
err := client.AddACL("/vms/100", "user@example.com", "PVEPowerAdmin", "user")
if err != nil {
    log.Fatal(err)
}
```

## Advanced

### Custom HTTP Client

```go
// With custom client
httpClient := &http.Client{
    Timeout: time.Second * 30,
}

client := proxmox.NewClient(
    "https://pve.example.com:8006/api2/json",
    proxmox.TokenCreds("root@pam", "token"),
    proxmox.WithHTTPClient(httpClient),
)
```

### With Logging

```go
client := proxmox.NewClient(
    "https://pve.example.com:8006/api2/json",
    proxmox.TokenCreds("root@pam", "token"),
    proxmox.WithDebug(true),
)
```

## Error Handling

```go
// Handle errors
_, err := client.StartVM("pve1", 999)
if err != nil {
    switch e := err.(type) {
    case *proxmox.APIError:
        fmt.Printf("API Error: %d - %s\n", e.Code, e.Message)
    default:
        fmt.Printf("Error: %s\n", e)
    }
}
```

## Keywords

#go #golang #api #client #automation

## Related Articles

- [[API]]
- [[API-Authentication]]
- [[API-Python-Client]]