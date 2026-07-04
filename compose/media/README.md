# Media Stack

## Stack

- Prowlarr
- Radarr
- Gluetun
- qBittorrent
- Jellyfin

## Layout

On mounted storage, currently `~/media`:

```text
downloads/        # downloader stages
  incomplete/
  complete/
    movies/

library/          # Jellyfin serves
  movies/
```

## Requirements

- Keep it simple.
- Put user-provided secrets in an untracked `.env` file, documented in `.env.example`.
- Use a VPN.
  - Generate a custom Gluetun OpenVPN config from VPNGate, then fail closed. See <https://github.com/rhew/private-torrent-downloader/blob/main/vpngate.py>.
  - qBittorrent external traffic must go through Gluetun.
  - Prowlarr external indexer traffic must go through Gluetun.
  - Radarr, Jellyfin, and local service-to-service traffic must stay on the normal Docker/LAN network.
- Follow this configuration priority order, best to worst:
  1. None: no config required
  2. Docker/Compose environment variables
  3. Mounted or added config files
  4. Automated service APIs
  5. Ansible automation beyond deployment
  6. CLI/UI/GUI procedures with instructions in "Manual Configuration" below
- Manage media:
  - Downloaders will import, rename, and move completed media to `library/`.
  - Downloaders will remove the completed torrent from qBittorrent after successful import.
- Choose UID/GID so qBittorrent, Radarr, and Jellyfin can read and write the expected paths.

### Networking

The biggest open item is how to force only Prowlarr and qBittorrent external traffic through Gluetun while keeping local service traffic normal. That is possible, but the Compose networking needs to be deliberate.

### Future

- Communicate failures, completed media, and similar events to Discord.
- Regenerate the Gluetun OpenVPN config and restart Gluetun on failure and on a cadence.
- Restart and repair failed services.
- Add services for downloading shows, books, and audiobooks.

### Out of Scope

- Serving media outside the local network

## Manual Configuration
