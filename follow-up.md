# Follow Up

## Service restart idempotence

Normal `site.yml` runs can restart unchanged services.

Current causes:

- `docker compose up -d` runs for Pi-hole on every normal run.
- `python3 vpngate.py` generates private-torrent-downloader VPN config on normal runs when the config is missing.
- `docker compose up -d --build` runs for private-torrent-downloader on every normal run.
- `docker build -t led-wall-weather .` runs on every normal run.
- `docker rm -f led-wall-weather` and `docker run -d ...` restart the LED wall client on every normal run.

Resolution direction:

- Use handlers so services restart only when managed config changes.
- Keep service startup available through explicit tags.
- Move `led-wall-weather` to Docker Compose or make its container management idempotent.
- Decide whether VPN Gate config generation belongs in normal convergence or an explicit private-torrent-downloader operation.
