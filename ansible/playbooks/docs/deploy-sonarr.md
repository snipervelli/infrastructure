# deploy-sonarr.yml

## Run

```bash
cd ~/infrastructure/ansible
ansible-playbook playbooks/deploy-sonarr.yml
```

## Purpose

Deploys Sonarr for TV-series management.

## Web Interface

```text
http://10.7.11.221:8989
```

## Persistent Data

```text
/opt/docker/data/sonarr
```

## Storage Mapping

Bethel is exposed to Sonarr as:

```text
/mnt/bethel -> /storage
```

Important paths:

```text
/storage/TV_Shows
/storage/Downloads
```

## Download Client

Sonarr connects to SABnzbd at:

```text
http://10.7.11.221:8080
```

Category:

```text
tv
```

Remote Path Mapping:

```text
Host:        10.7.11.221
Remote Path: /downloads
Local Path:  /storage/Downloads
```

## Indexers

Indexer management is centralized in Prowlarr.

Current validated indexers include:

```text
NZBGeek
NZBPlanet
```

## Quality Policy

Allowed:

```text
720p
1080p
```

Disallowed:

```text
2160p / 4K
Remux
```

Upgrades:

```text
Disabled
```

Maximum release size:

```text
3072 MB
```

## Monitoring Policy

Continuing series:

```text
Future episodes monitored
Past episodes unmonitored
```

Ended series:

```text
Left unmonitored by default
```

Validation performed:

```text
44 continuing series updated
71 future episodes monitored
465 ended series left untouched
No searches triggered
```

## End-to-End Validation

Validated workflow:

```text
Sonarr
   |
   v
Prowlarr
   |
   v
NZB indexer
   |
   v
SABnzbd
   |
   v
Gluetun / PIA
   |
   v
Bethel Downloads
   |
   v
Sonarr import
   |
   v
Bethel TV library
```

A Reacher episode was successfully downloaded and imported into the existing TV library.
