# deploy-prowlarr.yml

## Run

```bash
cd ~/infrastructure/ansible
ansible-playbook playbooks/deploy-prowlarr.yml
```

## Purpose

Deploys Prowlarr as the indexer-management service for the media automation stack.

## Network

Prowlarr uses the normal Docker host network path.

It is not routed through Gluetun by default.

## Persistent Data

```text
/opt/docker/data/prowlarr
```

## Web Interface

```text
http://10.7.11.221:9696
```

## Container Identity

```text
PUID=1000
PGID=100
```

## Role in Media Stack

```text
Prowlarr
   |
   +--> Sonarr
   |
   +--> Radarr
```

Prowlarr manages indexer configuration and synchronizes indexers to the media-management applications.
