# validate-base.yml

## Run

```bash
cd ~/infrastructure/ansible
ansible-playbook playbooks/validate-base.yml
```

## Purpose

`validate-base.yml` verifies that a managed Rocky Linux system meets the expected Linux baseline.

It is intended to validate the result of:

```text
base-linux.yml
```

The validation playbook should not make configuration changes.

## Target

Inventory group:

```ini
[rocky]
```

## Checks Performed

The playbook verifies the following services are running:

```text
sshd
chronyd
firewalld
qemu-guest-agent
```

It verifies the effective SSH configuration includes:

```text
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin without-password
```

or the equivalent:

```text
PermitRootLogin prohibit-password
```

It verifies SELinux is:

```text
Enforcing
```

It verifies the Ansible automation account has non-interactive sudo access.

It also verifies required troubleshooting commands are available, including:

```text
dig
ncat
tcpdump
traceroute
curl
tree
git
rsync
```

## Expected Result

A successful validation should end with:

```text
failed=0
```

and display:

```text
<hostname> passed the Linux baseline validation.
```

## Recommended Workflow

Apply the baseline:

```bash
ansible-playbook playbooks/base-linux.yml
```

Then validate it:

```bash
ansible-playbook playbooks/validate-base.yml
```

## Failure Handling

If validation fails, do not bypass the failed assertion merely to obtain a successful run.

Determine whether:

- the baseline role did not configure the setting
- another configuration file overrides the desired setting
- a required package is missing
- a required service is stopped
- the validation itself no longer reflects the intended baseline

Correct the underlying configuration or validation logic and rerun the playbook.

## Golden Image Workflow

Before finalizing a future golden image, the intended sequence is:

```bash
ansible-playbook playbooks/base-linux.yml
ansible-playbook playbooks/validate-base.yml
```

Only after validation succeeds should a dedicated golden-image candidate be finalized.
