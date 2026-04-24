# Pipeline Automation Scripts

## Overview

Pipeline automation creates VM environments on-demand. Useful for CI/CD, testing, and development workflows.

## Basic Pipeline

### Create Pipeline

```bash
#!/bin/bash
# create-env.sh - Create test environment

VM_NAME=$1
VM_TEMPLATE=${2:-ubuntu-template}

# Create VM
VUID=900
qm create $VUID \
  --name "$VM_NAME" \
  --clone "$VM_TEMPLATE" \
  --full 1

# Configure
qm set $VUID --memory 4096 --cores 2

# Start
qm start $VUID

echo "Created VM: $VUID"
```

### Destroy Pipeline

```bash
#!/bin/bash
# destroy-env.sh - Destroy test environment

VMID=$1

# Stop
qm stop $VMID --forceStop 1 || true

# Wait for stop
sleep 5

# Destroy
qm destroy $VMID

echo "Destroyed VM: $VMID"
```

## Build Pipeline

### Full CI/CD Pipeline

```bash
#!/bin/bash
# build-pipeline.sh - Full pipeline

set -e

VM_NAME="build-$RANDOM"
TEMPLATE="ubuntu-template"
VUID=910

echo "Creating build VM..."

# Create and start
qm clone $TEMPLATE $VUID --name "$VM_NAME" --full 1
qm start $VUID

# Wait for network
sleep 30

# Deploy application
echo "Deploying application..."

# Build
echo "Running build..."

# Test
echo "Running tests..."

# Cleanup
qm destroy $VUID

echo "Pipeline complete"
```

## GitLab CI Integration

```yaml
# .gitlab-ci.yml
variables:
  PVE_HOST: pve.example.com
  VM_TEMPLATE: ubuntu-template

.create:
  script:
    - ./create-vm.sh $CI_PIPELINE_ID $VM_TEMPLATE

.test:
  script:
    - ./run-tests.sh $VM_ID

build:
  stage: create
  script:
    - ./create-vm.sh ${CI_PIPELINE_ID}

cleanup:
  stage: cleanup
  script:
    - ./destroy-vm.sh $VM_ID
```

## Jenkins Pipeline

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    stages {
        stage('Create VM') {
            steps {
                sh './create-vm.sh'
            }
        }
        stage('Deploy') {
            steps {
                sh './deploy.sh'
            }
        }
        stage('Test') {
            steps {
                sh './test.sh'
            }
        }
        stage('Cleanup') {
            steps {
                sh './destroy-vm.sh'
            }
        }
    }
}
```

## Kubernetes Integration

### Scale on Demand

```bash
#!/bin/bash
# k8s-integration.sh

# Scale K8s -> Proxmox
for i in $(seq 1 $1); do
    qm clone template-$i vm-$i --full 1
    qm start vm-$i
done
```

## Keywords

#pipeline #cicd #automation #deployment #ci

## Related Articles

- [[Automation]]
- [[Scripts]]
- [[API]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
