# Service Inventory and Migration Plan

## Purpose

This document tracks the current homelab infrastructure, service placement, migration direction, and modernization priorities.

The goal is to avoid ad-hoc migrations and maintain a deliberate record of:

- current hosts
- current workloads
- dependencies
- migration risk
- target placement
- retirement candidates
- infrastructure modernization decisions

---

# Proxmox Hosts

## Citadel

### Role

Primary Proxmox virtualization host.

### Current Platform

```text
CPU:        Intel Xeon E5-2430
Cores:      6
Threads:    12
RAM:        94 GiB
Proxmox:    9.2
10GbE:      Intel X520 dual-port
1GbE:       Broadcom BCM5720 dual-port
```

### Storage

```text
local-zfs:
  mirrored 300 GB SAS disks
  approximately 78% utilized

vol1:
  approximately 2 TB Crucial BX500 SSD
  primary VM storage

vol2:
  approximately 512 GB Crucial M4 SSD

vmbackups:
  NFS backup target
```

### Current Running VMs

```text
100  mtgenesis
107  Plex
108  Content
120  test01
9001 ansible01
```

### Templates

```text
9000 tmpl-rocky98
```

### Direction

```text
KEEP FOR NOW
```

Citadel remains the primary virtualization host while the environment is modernized.

Long term, Citadel should be replaced by a substantially more efficient virtualization platform.

### Future Citadel Replacement Target

```text
Modern CPU architecture
8+ physical cores preferred
16+ threads
96 GB RAM minimum
128 GB+ preferred
10GbE minimum
Dual 10GbE desirable
Multiple NVMe devices
Strong Linux / Proxmox compatibility
IOMMU / PCI passthrough support
Low idle power
ECC preferred but not required
```

A replacement should provide a meaningful improvement in:

```text
performance
performance per watt
storage performance
networking
capacity
simplicity
```

---

## Phoenix

### Role

Secondary Proxmox virtualization host.

### Hardware

```text
System:      Dell OptiPlex 5080
CPU:         Intel Core i5-10600T
Cores:       6
Threads:     12
RAM:         16 GB DDR4
GPU:         Intel UHD 630
Network:     Intel I219-LM 1GbE
Storage:     512 GB NVMe VM storage
Boot disk:   120 GB SATA SSD
```

### Memory

Current configuration:

```text
2 x 8 GB SODIMM
16 GB total
```

Potential target:

```text
2 x 32 GB SODIMM
64 GB total
```

### Current VM

```text
200 exodus
```

`exodus` is a Plex experimentation VM and is not considered a long-term production workload.

### Direction

```text
KEEP
```

Phoenix is a strong candidate for:

```text
secondary Proxmox workloads
media workloads
Quick Sync transcoding
utility VMs
temporary migration workloads
Citadel maintenance landing zone
```

### Limitation

```text
1GbE networking
```

The system has no conventional PCIe expansion slot.

Do not turn Phoenix into an unreliable adapter experiment solely to obtain 10GbE.

---

# Automation Infrastructure

## ansible01

### Role

Primary Ansible control node.

### Platform

```text
Rocky Linux 9.8
IP: 10.7.11.211
```

### Toolchain

```text
Python:             3.12.13
ansible-core:       2.20.8
community.proxmox:  2.0.0
proxmoxer:          2.3.0
```

### Authentication Model

Personal administration:

```text
Mac
  -> mthomas@ansible01
  -> personal admin SSH key
```

Automation:

```text
ansible01
  -> ansible@managed-hosts
  -> ansible_ed25519
```

Proxmox API:

```text
ansible@pve!ansible01
```

API token secret is protected with Ansible Vault.

### Direction

```text
KEEP
```

---

# Golden Image and VM Provisioning

## Rocky Linux Template

```text
VMID: 9000
Name: tmpl-rocky98
OS:   Rocky Linux 9.8
```

Cloud-Init enabled.

Each clone receives unique:

```text
machine-id
SSH host keys
hostname
network configuration
SSH automation access
```

## VM Provisioning

Validated end-to-end provisioning workflow:

```text
provision-rocky-vm.yml
```

Workflow:

```text
Clone Proxmox template
Configure CPU and memory
Configure Cloud-Init networking
Start VM
Wait for SSH
Enroll SSH host key
Connect with Ansible
Apply base_linux
Validate host
```

---

# Docker Host

## mtgenesis

### Role

Current primary Docker/application host.

### Platform

```text
Rocky Linux 9.6
IP: 10.7.11.40
RAM: 16 GB
Disk: 180 GB
Docker: active
Tailscale: active
```

### Storage

Local application state resides primarily on the VM disk.

QNAP media storage:

```text
10.7.11.30:/Public
```

Mounted at:

```text
/qnap
```

Media Docker volumes reference:

```text
Music
Photos
Multimedia
TV_Shows
```

QNAP capacity observed:

```text
52 TB total
41 TB used
11 TB available
```

### Direction

```text
KEEP DURING MIGRATION
```

Do not perform a wholesale rebuild.

Services should be migrated deliberately into a reproducible Git-managed deployment.

---

# Active Docker Services

## Technitium DNS

```text
Container: dns01
Image: technitium/dns-server:latest
Compose: /opt/docker/technitium/docker-compose.yml
Persistent data: Docker volume technitium_config
```

### Importance

```text
CRITICAL
```

Current DNS address:

```text
10.7.11.40
```

This DNS service is currently used by infrastructure including Rocky Cloud-Init provisioning.

### Direction

```text
KEEP
MIGRATE CAREFULLY
```

No DNS migration should occur without a tested fallback.

---

## Traefik

```text
Image: traefik:latest
Compose: /opt/docker/traefik/docker-compose.yml
```

Persistent/configuration files:

```text
/opt/docker/traefik/data/traefik.yml
/opt/docker/traefik/data/config.yml
/opt/docker/traefik/data/acme.json
```

### Importance

```text
CRITICAL
```

### Direction

```text
REBUILD DECLARATIVELY
```

Future deployment should use controlled image versions and Git-managed configuration.

Secrets and ACME state must not be committed directly to Git.

---

## Gitea

```text
Container: gitea
Image: gitea/gitea:latest
Compose: /opt/docker/gitea/docker-compose.yml
```

Persistent data:

```text
/opt/docker/gitea/gitea/data
```

### Direction

```text
KEEP
MIGRATE AS APPLICATION STACK
```

---

## PostgreSQL

```text
Container: postgres
Image: postgres:16
Compose project: gitea
```

Persistent data:

```text
/opt/docker/gitea/postgres/data
```

### Direction

```text
KEEP
MIGRATE WITH GITEA
```

Gitea and PostgreSQL should be treated as a single migration unit.

---

## Home Assistant

```text
Image: lscr.io/linuxserver/homeassistant:latest
Compose: /opt/docker/homeassistant/docker-compose.yml
```

Persistent configuration/state includes:

```text
configuration.yaml
automations.yaml
scripts.yaml
scenes.yaml
secrets.yaml
.storage/
home-assistant_v2.db
```

### Importance

```text
HIGH
```

### Direction

```text
KEEP
MIGRATE CAREFULLY
```

Do not commit the full Home Assistant directory to Git.

Declarative configuration must be separated from:

```text
secrets
database
runtime state
authentication state
integration credentials
```

---

## Homebridge

```text
Image: homebridge/homebridge:latest
```

Active Compose definition:

```text
/data/compose/1/docker-compose.yml
```

Persistent state:

```text
/data/compose/1/volumes/homebridge
```

There is also a separate inactive-looking definition under:

```text
/opt/docker/homebridge
```

### Direction

```text
KEEP
REBUILD / NORMALIZE
```

Future deployment should be moved out of the Portainer-managed compose path.

---

## Jellyfin

```text
Image: jellyfin/jellyfin:latest
Compose: /opt/docker/jellyfin/docker-compose.yml
```

Persistent configuration under:

```text
/opt/docker/jellyfin/configs/
```

Media sources:

```text
movies
tv-shows
music
photos
```

Media originates from QNAP NFS storage.

### Direction

```text
KEEP
REBUILD
```

Potential future placement:

```text
Phoenix
```

because Phoenix provides Intel UHD 630 Quick Sync capability.

Final placement should consider network throughput and media workload.

---

## Tautulli

```text
Image: lscr.io/linuxserver/tautulli:latest
Compose: /opt/docker/tautulli/docker-compose.yml
Persistent state: /docker/tautulli
```

### Direction

```text
KEEP
REBUILD / NORMALIZE STORAGE PATH
```

---

## Uptime Kuma

```text
Image: louislam/uptime-kuma:1
Compose: /opt/docker/uptime-kuma/docker-compose.yml
Persistent volume: uptime-kuma_uptime-kuma
```

### Direction

```text
KEEP
REBUILD
```

Docker socket access should be reviewed.

---

## Portainer

```text
Image: portainer/portainer-ce:latest
Compose: /opt/docker/portainer/docker-compose.yml
```

Current data path appears to be:

```text
/home/username/portainer/data
```

An unused Docker volume also exists:

```text
portainer_portainer_data
```

### Direction

```text
KEEP DURING TRANSITION
```

Portainer is useful during migration but should not remain the authoritative configuration source.

Git-managed Compose should become authoritative.

---

## Watchtower

```text
Image: containrrr/watchtower
Compose: /opt/docker/watchtower/docker-compose.yml
Docker socket: RW
```

### Direction

```text
RETIRE
```

Future container updates should be deliberate and controlled through infrastructure automation rather than automatic unattended updates.

---

# Inactive / Retirement Candidates

## SABnzbd

```text
Compose: /opt/docker/sabnzbd/docker-compose.yml
Status: exited
Inactive approximately 12 months
```

### Direction

```text
INVESTIGATE
LIKELY REBUILD AS PART OF FUTURE MEDIA STACK
```

Do not assume the old stopped instance should be migrated directly.

---

## AAR / Gluetun

```text
Compose: /opt/docker/aar/docker-compose.yml
Container: gluetun
Status: exited
Inactive approximately 17 months
```

### Direction

```text
INVESTIGATE
LIKELY RETIRE OR REBUILD
```

---

## Old Images

Observed unused images include:

```text
qBittorrent
Prowlarr
deunhealth
```

### Direction

```text
DO NOT DELETE YET
RETIRE AFTER MIGRATION REVIEW
```

---

## Other Legacy Directories

Potential legacy or abandoned Docker content:

```text
/opt/docker/downloaders
/opt/docker/remote_dsktop
/opt/docker/nexus-docker
/opt/docker/nginx-docker
```

### Direction

```text
INVESTIGATE BEFORE REMOVAL
```

---

# Windows Legacy VMs

## Plex

```text
VMID: 107
OS: Windows
RAM: 16 GB
Storage: vol1
Network: vmbr2
```

### Direction

```text
MIGRATE / RETIRE
```

Do not lift-and-shift the Windows Plex VM unless a specific dependency requires it.

Preferred direction is a Linux/container-based media platform.

---

## Content

```text
VMID: 108
OS: Windows
RAM: 8 GB
Storage: local-zfs
Network: vmbr2
```

The VM currently consumes almost the entire `rpool/data` ZFS dataset.

### Direction

```text
MIGRATE / RETIRE
```

Determine remaining application/data dependencies before deletion.

---

# Temporary Systems

## test01

```text
VMID: 120
IP: 10.7.11.220
```

Used to validate automated Proxmox VM provisioning.

### Direction

```text
RETIRE WHEN NO LONGER NEEDED
```

---

# Migration Principles

## 1. Do Not Lift-and-Shift Everything

Where practical, workloads should be rebuilt using modern declarative configuration rather than copying legacy host state.

## 2. Git Becomes the Source of Truth

Future container definitions should be stored in version control.

Git should contain:

```text
Compose definitions
Ansible roles
deployment logic
documentation
non-secret configuration
```

Git must not contain:

```text
passwords
API keys
private SSH keys
database data
ACME private state
Home Assistant secrets
application databases
runtime state
```

## 3. Protect Critical Infrastructure

Special care is required for:

```text
Technitium DNS
Traefik
Gitea/PostgreSQL
Home Assistant
```

These services should have validated backups and rollback paths before migration.

## 4. Avoid Automatic Uncontrolled Upgrades

Important container image versions should eventually be pinned.

Watchtower should be retired once controlled deployment and update workflows are established.

## 5. Preserve Failure Domains

Do not consolidate every infrastructure and media service onto one physical node merely because capacity exists.

Phoenix, Citadel, and future hosts should be used to reduce correlated failure risk.

---

# Initial Migration Priorities

Suggested order:

```text
1. Establish service inventory and backups
2. Build Git-managed container deployment structure
3. Build Docker host baseline with Ansible
4. Migrate low-risk services first
5. Migrate Gitea/PostgreSQL
6. Rebuild media services
7. Evaluate Phoenix for Jellyfin/Plex
8. Migrate Home Assistant / Homebridge carefully
9. Migrate or redesign Traefik
10. Migrate Technitium DNS only with tested fallback
11. Retire legacy Windows VMs
12. Remove abandoned Docker stacks/images/data
```

---

# Current Status

```text
Host discovery:                 COMPLETE
Citadel discovery:              COMPLETE
Phoenix discovery:              COMPLETE
mtgenesis Docker discovery:     COMPLETE
Rocky golden template:          COMPLETE
Ansible baseline:               COMPLETE
Proxmox API automation:         COMPLETE
Automated Rocky provisioning:   COMPLETE

Docker migration automation:    NOT STARTED
Service migrations:             NOT STARTED
Citadel replacement:            FUTURE
```
