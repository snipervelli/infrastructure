# deploy-media-download.yml

## Run

```bash
cd ~/infrastructure/ansible
ansible-playbook playbooks/deploy-media-download.yml
```

## Purpose

Deploys the media download foundation consisting of:

```text
Gluetun
SABnzbd
```

SABnzbd shares Gluetun's network namespace so its Internet traffic is routed through Private Internet Access.

## Target

Inventory group:

```ini
[docker]
docker-test ansible_host=10.7.11.221
```

## VPN Architecture

```text
SABnzbd
   |
   v
Gluetun
   |
   v
Private Internet Access
   |
   v
Internet
```

SABnzbd does not receive an independent Docker network path.

If Gluetun is unavailable, SABnzbd cannot silently fall back to the Docker host's normal WAN connection.

## Credentials

PIA credentials are provided through Ansible Vault.

They are not stored in Git.

Runtime credentials are written to:

```text
/opt/docker/stacks/media-download/.env
```

with restricted permissions.

## Storage

SABnzbd configuration:

```text
/opt/docker/data/sabnzbd
```

Bethel NFS storage:

```text
/mnt/bethel
```

SABnzbd download mapping:

```text
/mnt/bethel/Downloads -> /downloads
```

Configured SABnzbd folders:

```text
/downloads/incomplete
/downloads/complete
```

Corresponding Bethel paths:

```text
/share/Public/Downloads/incomplete
/share/Public/Downloads/complete
```

## Media Identity

Containers use:

```text
PUID=1000
PGID=100
```

This aligns with the existing Bethel ownership model:

```text
UID 1000
GID 100
```

Validated resulting permissions:

```text
Downloads             uid=1000 gid=100 mode=770
Downloads/incomplete  uid=1000 gid=100 mode=770
Downloads/complete    uid=1000 gid=100 mode=770
```

## SABnzbd Web Interface

SABnzbd is exposed through Gluetun at:

```text
http://10.7.11.221:8080
```

The port is published by Gluetun because SABnzbd shares Gluetun's network namespace.

## Usenet

Current provider:

```text
UsenetServer
```

Recommended encrypted NNTP connection:

```text
Host: news.usenetserver.com
SSL:  enabled
Port: 563
```

Provider credentials are configured inside SABnzbd and should not be committed to Git.

## Validation Completed

The following tests have passed:

```text
PIA tunnel established
Docker host retained normal WAN IP
Gluetun traffic exited through PIA
SABnzbd traffic exited through PIA
VPN restart produced a new PIA exit IP
No normal WAN leakage was observed
Bethel NFS mounted read/write
SABnzbd created download directories successfully
100 MB SABnzbd test download completed
```

Validated download flow:

```text
UsenetServer
      |
      v
PIA
      |
      v
Gluetun
      |
      v
SABnzbd
      |
      v
/downloads/incomplete
      |
      v
processing
      |
      v
/downloads/complete
      |
      v
Bethel
```

## Current Status

```text
Gluetun: validated
PIA: validated
SABnzbd: validated
Bethel download storage: validated
```

## Next Steps

The next media services will be added separately:

```text
Prowlarr
Sonarr
Radarr
```

These services will not automatically share Gluetun's network namespace.

Existing media libraries will not be modified until the automation workflow is independently validated.
