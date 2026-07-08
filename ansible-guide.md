# Ansible Guide

Ansible will make my servers reproducible.

## Scope

- Harden the host after the [install runbook](server-init-runbook.md).
- Install and update Docker and Docker Compose.
- Clone service projects into `/home/rhew`.
- Deploy required project config and credentials without checking secrets into this repo.

## Generic Docker Host

Ansible will manage:

- Docker and Docker Compose packages
- `lazydocker`
- Docker daemon config
- Docker access for `rhew`
- weekly Docker pruning for unused images, stopped containers, and build cache older than 7 days
- Neovim as `vim` and `nvim` with `rhew`'s config
- project checkout paths
- project config deployment

## Host Hardening

Ansible will manage:

- SSH: keep password authentication and root login disabled.
- Packages: apply security updates automatically.
- Reboots: automatically reboot after security updates when required.
- Docker logs: cap container log size.
- Data paths: set expected owner, group, and mode.

Out of scope:

- host firewall policy
- Docker rootless mode
- Docker user namespace remapping

## Home Server Specifics:

- committed defaults live in `host_vars/lenny.yml`
- host-local or private overrides live in ignored `host_vars/lenny.local.yml`
- mount the media drive at `/home/rhew/media`
- create `/home/rhew/media/downloads/complete`
- create `/home/rhew/media/downloads/incomplete`
- create `/home/rhew/media/library`
- create `/home/rhew/pi-hole/etc-pihole`
- create `/home/rhew/pi-hole/etc-dnsmasq.d`
- create `/opt/private-torrent-downloader`
- Services:
   - `https://github.com/rhew/pi-hole.git`
   - `https://github.com/rhew/led-pixel-wall.git`
   - `https://github.com/rhew/agent-control-plane.git`
   - selected runtime files from the local peer checkout `../private-torrent-downloader` copied into `/opt/private-torrent-downloader`
- `led-pixel-wall` config:
   - defined in `host_vars/lenny.local.yml`
- Media VPNGate maintenance:
   - `private-torrent-downloader-vpn-refresh.timer` refreshes the generated Gluetun OpenVPN config daily.
   - The timer does scheduled refresh only, not healthcheck-based repair.
   - App config is rendered from Ansible vars; do not edit `/opt/private-torrent-downloader/.env` directly.

## Web Server Specifics:

- Services:
   - `https://github.com/rhew/rhew.org.git`

## Open Inputs

- Required config files for each project
- Required credentials for each project
- How Ansible will deliver config and credentials without committing secrets
