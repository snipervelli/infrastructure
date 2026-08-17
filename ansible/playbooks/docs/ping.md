# ping.yml

## Run

```bash
cd ~/infrastructure/ansible
ansible-playbook playbooks/ping.yml
```

## Purpose

`ping.yml` performs a basic Ansible connectivity test against managed systems.

Use this playbook before applying configuration when verifying that a new host is reachable and correctly configured for Ansible management.

## What It Tests

The playbook verifies:

- Ansible can reach the target
- SSH or the configured connection method works
- Python is available on the managed host
- privilege and account configuration permit Ansible operation
- Ansible facts can be gathered
- the target hostname can be reported

## Expected Result

A successful host should report:

```text
ok
```

for the connectivity test.

The play recap should contain:

```text
unreachable=0
failed=0
```

## New VM Workflow

For a newly provisioned VM, use this playbook before applying the baseline:

```bash
ansible-playbook playbooks/ping.yml
```

Once connectivity succeeds:

```bash
ansible-playbook playbooks/base-linux.yml
ansible-playbook playbooks/validate-base.yml
```

## Troubleshooting

If the host reports:

```text
UNREACHABLE
```

check:

- VM power state
- IP address
- DNS resolution
- network routing
- firewalld
- SSH service
- SSH public key
- Ansible inventory
- Ansible remote user
- SSH host-key changes after rebuilding or cloning a VM

## Important

Ansible `ping` is not the same as ICMP `ping`.

The Ansible ping module tests whether Ansible can connect to and execute its module on the target system.
