# base-linux.yml

## Run

```bash
cd ~/infrastructure/ansible
ansible-playbook playbooks/base-linux.yml
```

## Purpose

`base-linux.yml` applies the standard Linux configuration baseline to managed Rocky Linux systems.

The playbook is designed to be idempotent and may be run repeatedly.

## Target

Inventory group:

```ini
[rocky]
```

Example:

```ini
[rocky]
ansible01 ansible_connection=local
```

## Role

The playbook applies:

```text
roles/base_linux
```

## Actions Performed

The baseline currently:

- installs standard administration packages
- installs network troubleshooting utilities
- installs Cloud-Init
- installs and enables the QEMU guest agent
- enables SSH
- enables Chrony
- enables firewalld
- ensures SELinux is enforcing
- configures SSH public-key authentication
- disables SSH password authentication
- prevents root password-based SSH login
- configures passwordless sudo for the Ansible automation account

## SSH Policy

The role manages:

```text
/etc/ssh/sshd_config.d/10-erwinsolutions-hardening.conf
```

with:

```text
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin prohibit-password
```

The `10-` prefix intentionally places the policy before the Cloud-Init SSH configuration.

## Validation

After applying the baseline, run:

```bash
cd ~/infrastructure/ansible
ansible-playbook playbooks/validate-base.yml
```

## Idempotence Test

Run the baseline twice:

```bash
ansible-playbook playbooks/base-linux.yml
ansible-playbook playbooks/base-linux.yml
```

After the system is already configured, the second run should normally report:

```text
changed=0
failed=0
```

## When To Run

Run this playbook:

- after provisioning a new Rocky Linux VM
- after rebuilding a server
- after changing the base Linux role
- periodically when verifying configuration consistency

## Important

This is a normal configuration-management playbook.

Unlike `golden-finalize.yml`, it does not intentionally remove machine identity or prepare a system for template conversion.
