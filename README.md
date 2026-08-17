# Infrastructure

Infrastructure-as-code and automation for the Erwin Solutions homelab.

## Repository Structure

### `ansible/`

Configuration management for Linux systems.

Includes:

- Base Linux configuration
- Security baseline
- Package management
- Service configuration
- Validation playbooks
- Golden image preparation
- Future application and infrastructure roles

### `proxmox/`

Proxmox VE automation.

Planned content includes:

- VM provisioning scripts
- Golden image creation
- Template management
- Cloud-Init configuration
- VM lifecycle automation

### `kickstart/`

Automated operating system installation.

Planned content includes:

- Rocky Linux Kickstart configurations
- Unattended golden image builds

### `docs/`

Infrastructure documentation, architecture notes, procedures, and recovery information.

## Current Platform

The initial automation targets Rocky Linux 9 and Proxmox VE.

## Security

Secrets, private SSH keys, passwords, and environment-specific credentials must not be committed to this repository.
