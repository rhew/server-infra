# Local Media Stack Development

These steps are for testing the Compose stack directly from this repo. Normal deployment to `lenny` should use Ansible from the main README.

Create a local `.env`:

```bash
cp .env.example .env
```

Generate a VPNGate OpenVPN config and start the stack:

```bash
./vpngate.py
docker compose up -d
```

`./vpngate.py` writes `gluetun/vpngate.ovpn` and `gluetun/vpngate-state.json`, which are intentionally ignored by git.

Refresh the VPNGate endpoint and recreate Gluetun locally with:

```bash
./media-vpn-refresh
```
