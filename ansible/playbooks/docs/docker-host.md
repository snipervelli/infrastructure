# docker-host.yml

## Run

```bash
cd ~/infrastructure/ansible
ansible-playbook playbooks/docker-host.yml
```

## Purpose

Configures managed Rocky Linux systems as standardized Docker application hosts.

## Target

Inventory group:

```ini
[docker]
```

Example:

```ini
[docker]
docker-test ansible_host=10.7.11.221
```

## Roles Applied

```text
base_linux
docker_host
```

## Actions Performed

The playbook:

- applies the standard Rocky Linux baseline
- installs Docker CE
- installs the Docker Compose plugin
- enables and starts Docker
- creates standard Docker filesystem paths
- optionally configures QNAP NFS storage
- configures required firewall services
- validates Docker and Docker Compose

## Standard Directory Layout

```text
/opt/docker/
├── stacks
├── data
└── backups
```

## Optional QNAP Mount

The Docker host role can optionally mount:

```text
10.7.11.30:/Public
```

at:

```text
/mnt/qnap
```

Enable with:

```yaml
docker_host_qnap_mount_enabled: true
```

Default mount options:

```text
rw,hard,nfsvers=4.1,_netdev
```

## Validation

The role verifies:

```bash
docker --version
docker compose version
docker info
```

## Safety

Do not initially target the existing `mtgenesis` production Docker host.

Test this role against a newly provisioned Rocky VM first.

## Intended Migration Workflow

```text
Provision clean Rocky VM
        |
        v
base_linux
        |
        v
docker_host
        |
        v
Deploy low-risk Git-managed stacks
        |
        v
Migrate services from mtgenesis gradually
```

## Current Test Host

```text
Name: docker-test
IP:   10.7.11.221
```

## Expected Result

A successful run should end with:

```text
failed=0
unreachable=0
```

and Docker should be active and ready for application stacks.

## Next Steps

After validating `docker-test`, deploy low-risk services first.

Recommended early candidates:

```text
Uptime Kuma
Tautulli
Portainer
```

Do not migrate critical services such as:

```text
Technitium DNS
Traefik
Gitea/PostgreSQL
Home Assistant
```

until their backup and rollback plans are documented and tested.
