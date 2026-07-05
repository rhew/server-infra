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
  - Generate a custom Gluetun OpenVPN config from VPNGate with `./vpngate.py`, then fail closed.
  - qBittorrent external traffic must go through Gluetun.
  - Prowlarr external indexer traffic uses the normal WAN for now.
  - Prowlarr, Radarr, Jellyfin, and local service-to-service traffic must stay on the normal Docker/LAN network.
- Follow this configuration priority order, best to worst:
  1. None: no config required
  2. Docker/Compose environment variables
  3. Mounted or added config files
  4. Automated service APIs
  5. Ansible automation beyond deployment
  6. CLI/UI/GUI procedures with instructions in "Manual Configuration" below
- Manage media:
  - Radarr will import, rename, and move completed media to `library/`.
  - Downloaders will remove the completed torrent from qBittorrent after successful import.
- Choose UID/GID so qBittorrent, Radarr, and Jellyfin can read and write the expected paths.

### Networking

Target design:

- Gluetun owns the VPN connection, using a generated VPNGate OpenVPN config.
- qBittorrent shares Gluetun's network namespace with `network_mode: "service:gluetun"`.
  - qBittorrent ports are published on the Gluetun service, not the qBittorrent service.
  - Gluetun's firewall is the fail-closed boundary.
- Prowlarr stays on the normal Docker network.
  - Prowlarr indexer traffic uses the normal WAN.
  - This keeps Prowlarr simple and avoids proxy configuration until there is a clear need for it.
- Radarr stays on the normal Docker network.
  - Radarr talks to Prowlarr over the normal Docker network.
  - Radarr talks to qBittorrent at `gluetun:8080`, because qBittorrent shares Gluetun's network namespace.
  - Radarr imports completed downloads from `downloads/` and writes managed media to `library/`.
- Jellyfin stays on host networking if that remains the simplest way to support DLNA discovery.
  - Jellyfin mounts `library/` read/write so it can store local images and metadata next to media files.
- Expose admin UIs on the LAN to start.
  - Goal: remove LAN UI exposure once access patterns are verified and frequent direct access is not required.

Torrent inbound port choice:

- Publishing an inbound torrent port through Gluetun can improve peer connectivity and upload/download performance.
- The risk is extra exposed surface on the VPN endpoint and more configuration churn, especially with VPNGate because endpoints are volatile and may not support stable forwarded ports.
- Start outbound-only unless qBittorrent performance is poor or seeding/connectivity requirements demand inbound reachability.

### Future

- Communicate failures, completed media, and similar events to Discord.
- Regenerate the Gluetun OpenVPN config and restart Gluetun on failure and on a cadence.
- Restart and repair failed services.
- Add services for downloading shows, books, and audiobooks.
- Automate Prowlarr first-run credentials and a deterministic indexer allowlist.

### Out of Scope

- Serving media outside the local network

## Deployment

Ansible deploys this stack to `media_stack_dir`, currently `/opt/media-stack` on `lenny`. The remote host does not need a checkout of this repo.

The role copies these files into the managed directory:

- `docker-compose.yml`
- `vpngate.py`
- `media-vpn-refresh`

The role renders `.env` from Ansible vars in `host_vars/lenny.yml` and `host_vars/lenny.local.yml`. Do not edit files directly on `lenny`; update Ansible vars and rerun:

```bash
ansible-playbook playbooks/site.yml --tags media_stack
```

Open the service UIs on the LAN:

- qBittorrent: `http://lenny:8080`
- Prowlarr: `http://lenny:9696`
- Radarr: `http://lenny:7878`
- Jellyfin: `http://lenny:8096`

Check the qBittorrent container log for the initial Web UI credentials before logging in:

```bash
ssh lenny 'docker logs --tail 200 media-qbittorrent'
```

Use the temporary `admin` password printed in the log, then set a permanent password in qBittorrent preferences.

On first login to Prowlarr, create the initial admin credentials before configuring indexers.

Then add at least one indexer in Prowlarr. The "No indexers found" message is expected until you do this.

Local-only Compose usage is documented in [LOCAL_DEV.md](LOCAL_DEV.md).

## VPNGate Maintenance

The generator reads optional VPNGate filters from the rendered `.env`:

```env
VPNGATE_COUNTRY=
VPNGATE_MIN_SPEED=5000000
VPNGATE_MAX_PING=300
VPNGATE_MIN_UPTIME=86400
```

`./vpngate.py` filters candidates, validates the decoded OpenVPN config, writes it atomically, and records the selected endpoint in `gluetun/vpngate-state.json`.

Refresh the VPNGate endpoint and recreate Gluetun manually with:

```bash
./media-vpn-refresh
```

Ansible installs and enables `media-vpn-refresh.timer`, which runs `media-vpn-refresh.service` daily with a randomized delay from the managed stack directory. This is scheduled maintenance only; it does not perform healthcheck-based repair.

## Manual Configuration

### Service URLs

- qBittorrent: `http://lenny:8080`
- Prowlarr: `http://lenny:9696`
- Radarr: `http://lenny:7878`
- Jellyfin: `http://lenny:8096`

### Radarr

1. Open `http://lenny:7878`.
2. Complete the initial admin credential setup on first login.
3. Configure qBittorrent as the download client:

```text
Settings -> Download Clients -> + -> qBittorrent
```

   Fill out these fields:

   ```text
   Host: gluetun
   Port: 8080
   Username: admin
   Password: use the temporary password printed in the qBittorrent log
   Category: movies
   ```
4. Configure the movie root folder:

```text
Settings -> Media Management -> Root Folders
```

   Set the movie library path to `/library/movies`.

5. When importing a file, keep the release tag in the final name instead of stripping it. Use the movie title and year as the base name, then append any descriptive edition or version tag that belongs to the release.

   Radarr should rename and move the file into `/library/movies` with that descriptive suffix intact.

### Prowlarr

1. Open `http://lenny:9696`.
2. Create the initial admin credentials on first login.
3. Add `LimeTorrents` as an indexer.
4. Enable it for movies only.
5. Leave the other indexers out for now.

### Jellyfin

1. Open `http://lenny:8096`.
2. Finish the first-run setup.
3. Add `/media/movies` as the movie library path.
   a. Automatically add to collection
   b. Save artwork into media folders
4. Enable Intel hardware transcoding in the admin dashboard:

   ```text
   Dashboard -> Playback -> Transcoding
   ```

   Enable hardware acceleration and select the Intel VA-API / Quick Sync option that Jellyfin offers for `/dev/dri`.
