# golden-finalize.yml

## Run

```bash
cd ~/infrastructure/ansible

ansible-playbook playbooks/golden-finalize.yml \
  -e golden_finalize_confirm=YES-I-UNDERSTAND
```

> **WARNING:** This is a destructive playbook. Run it only against a VM specifically intended to become a golden image/template.

## Purpose

`golden-finalize.yml` prepares a Linux VM for conversion into a Proxmox golden template.

It removes machine-specific identity and state that must not be duplicated when the template is cloned.

This playbook is intentionally separate from `base-linux.yml`.

`base-linux.yml` configures normal servers.

`golden-finalize.yml` generalizes a VM immediately before it becomes a template.

## Target Inventory Group

This playbook only targets hosts in the:

```ini
[golden]
```

inventory group.

Example:

```ini
[golden]
golden-test ansible_host=10.7.11.213
```

Do not place normal production servers in this group.

## Required Confirmation

The role refuses to perform finalization unless this exact variable is supplied:

```text
golden_finalize_confirm=YES-I-UNDERSTAND
```

Therefore this:

```bash
ansible-playbook playbooks/golden-finalize.yml
```

should fail safely.

The destructive run requires:

```bash
ansible-playbook playbooks/golden-finalize.yml \
  -e golden_finalize_confirm=YES-I-UNDERSTAND
```

## Safety Controls

The playbook currently has two primary safety controls.

### 1. Dedicated inventory group

Only systems explicitly assigned to `[golden]` are targeted.

### 2. Explicit confirmation variable

The role requires:

```text
YES-I-UNDERSTAND
```

before generalization can begin.

Both protections are intentional.

## Actions Performed

The `golden_finalize` role performs the following operations.

### Validate SSH

Checks the SSH server configuration before SSH host keys are removed.

### Validate Cloud-Init

Confirms that Cloud-Init is installed before attempting to clean its state.

### Validate automation account

Confirms that the configured Ansible automation user exists.

Default:

```text
ansible
```

### Remove authorized_keys

Removes:

```text
/home/ansible/.ssh/authorized_keys
```

The SSH public key should be injected into future clones by Proxmox Cloud-Init instead of being permanently baked into the template.

### Clean Cloud-Init

Runs:

```bash
cloud-init clean --logs --machine-id
```

This removes instance-specific Cloud-Init state and prepares Cloud-Init to treat the next boot as a new instance.

### Reset machine identity

The machine ID is reset so clones do not inherit the same Linux machine identity.

Expected final state is either:

```text
uninitialized
```

or an empty machine-id.

### Remove SSH host keys

Existing files matching:

```text
/etc/ssh/ssh_host_*
```

are removed.

Each clone should generate unique SSH host keys on first boot.

### Verify cleanup

The role verifies that:

- machine-id was reset
- SSH host keys were removed

### Shutdown

By default the role powers the VM off after successful finalization.

The powered-off VM can then be converted into a Proxmox template.

## Default Variables

Defaults are located at:

```text
roles/golden_finalize/defaults/main.yml
```

Current defaults:

```yaml
golden_finalize_confirm: ""

golden_finalize_ansible_user: ansible

golden_finalize_remove_authorized_keys: true

golden_finalize_remove_ssh_host_keys: true

golden_finalize_clean_cloud_init: true

golden_finalize_shutdown: true
```

## Syntax Check

Before running the playbook:

```bash
cd ~/infrastructure/ansible

ansible-playbook --syntax-check playbooks/golden-finalize.yml
```

A warning about the `golden` host pattern is expected if no `[golden]` inventory group currently exists.

## Safety Test

Before performing a real finalization, test the confirmation guard.

Run:

```bash
cd ~/infrastructure/ansible

ansible-playbook playbooks/golden-finalize.yml
```

If a host exists in `[golden]`, the playbook should stop at the confirmation assertion because the required confirmation variable was not provided.

## Destructive Test

Only use a disposable VM specifically created to test golden-image finalization.

Example inventory:

```ini
[golden]
golden-test ansible_host=10.7.11.213
```

First verify connectivity:

```bash
cd ~/infrastructure/ansible

ansible golden -m ping
```

Then perform the finalization:

```bash
ansible-playbook playbooks/golden-finalize.yml \
  -e golden_finalize_confirm=YES-I-UNDERSTAND
```

The SSH connection will eventually disappear because the VM is intentionally powered off.

## Expected Final State

After successful execution:

```text
Cloud-Init state     cleaned
Machine ID           uninitialized/empty
SSH host keys        removed
Ansible authorized key removed
VM                    powered off
```

The VM is then ready for conversion into a Proxmox template.

## Proxmox Workflow

The intended workflow is:

```text
Rocky Linux installation
        |
        v
base-linux.yml
        |
        v
validate-base.yml
        |
        v
Golden image candidate
        |
        v
golden-finalize.yml
        |
        v
VM powers off
        |
        v
Convert VM to Proxmox template
        |
        v
Clone template
        |
        v
Proxmox Cloud-Init supplies:
  - hostname
  - network configuration
  - DNS
  - SSH public key
  - automation user
        |
        v
Unique Rocky Linux VM
```

## Important

Never run this playbook against:

- the Ansible control node
- an existing production server
- Docker hosts containing production workloads
- Proxmox hosts
- any VM that has not been explicitly designated as a golden-image candidate

When in doubt, do not run the playbook.
