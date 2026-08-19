# deploy-gluetun.yml

## Run

```bash
cd ~/infrastructure/ansible
ansible-playbook playbooks/deploy-gluetun.yml
```

## Purpose

Deploys Gluetun as the VPN gateway for selected Docker workloads.

## VPN Provider

```text
Private Internet Access
```

Credentials are provided through Ansible Vault and are not stored in Git.

## Deployment Paths

Compose:

```text
/opt/docker/stacks/gluetun/compose.yml
```

Persistent state:

```text
/opt/docker/data/gluetun
```

Runtime secrets:

```text
/opt/docker/stacks/gluetun/.env
```

The `.env` file exists only on the Docker host and must not be committed.

## Initial Scope

Gluetun is deployed by itself first.

SABnzbd will not be attached until the following have been validated:

```text
VPN tunnel establishes successfully
Public IP differs from the Docker host
Traffic through Gluetun exits through PIA
Kill switch prevents fallback traffic
```

## Intended Network Model

```text
SABnzbd
   |
   v
Gluetun
   |
   v
PIA
   |
   v
Internet
```

Other services such as Sonarr and Radarr will not automatically share the VPN namespace.
