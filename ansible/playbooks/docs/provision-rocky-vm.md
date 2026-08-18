# provision-rocky-vm.yml

## Run

```bash
cd ~/infrastructure/ansible

ansible-playbook playbooks/provision-rocky-vm.yml \
  -e vmid=120 \
  -e vm_name=test01 \
  -e vm_ip=10.7.11.220/24
```

## Optional Overrides

```bash
ansible-playbook playbooks/provision-rocky-vm.yml \
  -e vmid=120 \
  -e vm_name=test01 \
  -e vm_ip=10.7.11.220/24 \
  -e vm_cores=4 \
  -e vm_memory=8192
```

## Purpose

`provision-rocky-vm.yml` creates a new Rocky Linux VM from the Proxmox golden template and prepares it for immediate Ansible management.

The playbook is intended to be the normal day-to-day workflow for provisioning new Rocky Linux servers.

## Current Template

Default Proxmox template VMID:

```text
9000
```

Template name:

```text
tmpl-rocky98
```

## Required Variables

The following values must be supplied:

```text
vmid
vm_name
vm_ip
```

Example:

```text
vmid=120
vm_name=test01
vm_ip=10.7.11.220/24
```

## Defaults

Unless overridden, the playbook uses:

```text
CPU cores:       2
Memory:          4096 MiB
Storage:         vol1
Gateway:         10.7.11.1
DNS:             10.7.11.40
Search domain:   erwinsolutions.com
Cloud-Init user: ansible
Proxmox node:    Citadel
Template VMID:   9000
```

## Workflow

The playbook performs the following actions.

### 1. Validate input

Confirms the required variables are present:

```text
vmid
vm_name
vm_ip
```

### 2. Check Proxmox for VMID conflicts

Queries the Proxmox API and refuses to continue if the requested VMID already exists.

The playbook does not overwrite an existing VM.

### 3. Clone the Rocky template

Clones:

```text
tmpl-rocky98
```

from VMID:

```text
9000
```

to the requested new VMID.

### 4. Configure VM resources

Applies the requested or default:

```text
CPU cores
Memory
Storage
```

### 5. Configure Cloud-Init

Configures:

```text
Static IP
Gateway
DNS
Search domain
Cloud-Init automation user
```

The new VM receives its SSH automation key from the Proxmox template Cloud-Init configuration.

### 6. Start the VM

Starts the new Proxmox VM.

### 7. Wait for SSH

Waits for TCP port 22 to become reachable.

### 8. Register the SSH host key

The playbook:

- removes any stale SSH host key for the requested IP
- scans the new VM's SSH host keys
- stores the current host keys in the Ansible controller's `known_hosts`

Host-key verification remains enabled.

### 9. Add the VM to temporary Ansible inventory

The new host is added dynamically to:

```text
newly_provisioned
```

using:

```text
User: ansible
SSH key: ~/.ssh/ansible_ed25519
```

### 10. Wait for Ansible connectivity

The playbook confirms actual SSH/Ansible connectivity before configuration begins.

### 11. Apply the Linux baseline

Applies:

```text
roles/base_linux
```

This includes:

- standard administration packages
- network troubleshooting packages
- Cloud-Init
- QEMU guest agent
- SSH hardening
- SELinux enforcing
- firewalld
- Chrony
- passwordless sudo for the Ansible automation account

### 12. Validate the resulting VM

The provisioning workflow verifies:

- SSH is running
- Chrony is running
- firewalld is running
- QEMU guest agent is running
- SSH password authentication is disabled
- public-key authentication is enabled
- root password SSH login is disabled
- SELinux is enforcing
- non-interactive sudo works

## Successful Result

A successful run ends with a message similar to:

```text
test01 provisioned successfully at 10.7.11.220.
```

The play recap should show:

```text
unreachable=0
failed=0
```

## Verified End-to-End Test

The provisioning workflow has been successfully tested with:

```text
VMID:     120
Name:     test01
IP:       10.7.11.220/24
Template: tmpl-rocky98
```

The test successfully completed:

```text
Template clone
Cloud-Init configuration
VM startup
SSH host-key enrollment
Ansible SSH connectivity
base_linux application
SSH validation
SELinux validation
sudo validation
```

## Safety

The playbook currently refuses to continue when the requested VMID already exists.

It does not automatically destroy, replace, or overwrite an existing VM.

Before provisioning, ensure the requested IP address is not already assigned to another system.

## Authentication

### Proxmox

The playbook uses the dedicated Proxmox API identity:

```text
ansible@pve
```

with the API token:

```text
ansible01
```

The API token secret is stored in an encrypted Ansible Vault.

### Managed VM

The Ansible controller connects to the new VM using:

```text
User: ansible
Key:  ~/.ssh/ansible_ed25519
```

## Python / Ansible Environment

The provisioning workflow uses the dedicated infrastructure virtual environment:

```text
~/.venvs/infrastructure
```

Current validated toolchain:

```text
Python:             3.12.13
ansible-core:       2.20.8
community.proxmox:  2.0.0
proxmoxer:          2.3.0
```

The Ansible user's PATH is configured so the infrastructure environment is used automatically.

## Troubleshooting

If provisioning fails during the Proxmox phase, check:

- Proxmox API connectivity
- API token secret
- API token ACL permissions
- `SDN.Use` permission for `vmbr0`
- template VMID 9000
- storage `vol1`
- requested VMID

If provisioning fails during SSH setup, check:

- Cloud-Init completion
- IP configuration
- SSH service
- SSH host-key state
- `~/.ssh/known_hosts` permissions
- `~/.ssh/ansible_ed25519`
- automation user's `authorized_keys`

If configuration fails after SSH succeeds, run:

```bash
ansible-playbook playbooks/base-linux.yml
```

and then:

```bash
ansible-playbook playbooks/validate-base.yml
```

against the affected managed host.

## Related Playbooks

```text
ping.yml
base-linux.yml
validate-base.yml
golden-finalize.yml
proxmox-api-test.yml
```
