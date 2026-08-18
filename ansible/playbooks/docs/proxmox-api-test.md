# proxmox-api-test.yml

## Run

```bash
cd ~/infrastructure/ansible

ansible-playbook playbooks/proxmox-api-test.yml \
  --ask-vault-pass
```

## Purpose

Tests read-only authentication from the Ansible control node to the Proxmox VE API.

This playbook does not create, modify, start, stop, or delete virtual machines.

## Target

The playbook runs locally on the Ansible control node.

It connects to the Proxmox API defined in:

```text
group_vars/proxmox/main.yml
```

## Authentication

The Proxmox API token secret is stored encrypted in:

```text
group_vars/proxmox/vault.yml
```

The vault password is requested interactively when the playbook runs.

## Expected Result

A successful run should report a message similar to:

```text
Proxmox API authentication succeeded and returned 12 cluster resources.
```

The exact number of resources will vary.

## Safety

This is a read-only API test.

It uses:

```text
GET /api2/json/cluster/resources
```

and does not modify Proxmox.

## Troubleshooting

If authentication fails, verify:

- Proxmox API host
- API user
- token ID
- token ACL permissions
- vault secret
- network connectivity to TCP 8006
- certificate validation setting
