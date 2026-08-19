# deploy-uptime-kuma.yml

## Run

```bash
cd ~/infrastructure/ansible
ansible-playbook playbooks/deploy-uptime-kuma.yml
```

## Purpose

Deploys Uptime Kuma to hosts in the `docker` inventory group using the Git-managed Compose definition.

## Target

```ini
[docker]
docker-test ansible_host=10.7.11.221
```

## Repository Source

```text
containers/stacks/uptime-kuma/compose.yml
```

## Host Deployment Path

```text
/opt/docker/stacks/uptime-kuma/compose.yml
```

## Persistent Data

```text
/opt/docker/data/uptime-kuma
```

The persistent application state is intentionally separate from the Compose definition.

Deleting or recreating the container should not remove Uptime Kuma data.

## Published Port

```text
3001/tcp
```

Example:

```text
http://10.7.11.221:3001
```

## Deployment Workflow

```text
Git Compose definition
        |
        v
Ansible
        |
        v
/opt/docker/stacks/uptime-kuma
        |
        v
docker compose up -d
        |
        v
Persistent state
/opt/docker/data/uptime-kuma
```

## Validation

The playbook verifies:

```text
Compose configuration is valid
Container is started
HTTP responds on port 3001
docker compose ps succeeds
```

## Persistence Test

After initial deployment:

```bash
docker compose down
docker compose up -d
```

The Uptime Kuma configuration should remain because data is stored outside the container.

## Safety

This deployment targets `docker-test`.

It does not modify the existing Uptime Kuma instance on `mtgenesis`.
