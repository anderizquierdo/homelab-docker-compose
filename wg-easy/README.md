# 🔒 WireGuard Easy (wg-easy)

wg-easy is a **WireGuard VPN server** with a built-in web UI for managing clients. It simplifies WireGuard setup and peer management — generating QR codes and config files for clients without needing to touch config files manually.

## Services

### `wg-easy`

- **Image**: [`ghcr.io/wg-easy/wg-easy:latest`](https://ghcr.io/wg-easy/wg-easy)
- Runs the WireGuard kernel module and a Node.js web UI for peer management.

## Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| `51820` | UDP | WireGuard VPN tunnel — must be reachable from the internet |
| `51821` | TCP | wg-easy web UI |

## Volumes

| Volume | Container path | Purpose |
|--------|---------------|---------|
| `etc_wireguard` *(named)* | `/etc/wireguard` | WireGuard configuration, private/public keys, and peer configs |

## Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| `PASSWORD_HASH` | *(from `.env`)* | Bcrypt hash of the web UI password. Generate with: `docker run --rm -it ghcr.io/wg-easy/wg-easy wgpw YOUR_PASSWORD` |
| `WG_HOST` | *(from `.env`)* | Public hostname or IP of the server (e.g. `vpn.yourdomain.com`) — without `http://` or `https://` |
| `PORT` | `51821` | Web UI port |
| `WG_PORT` | `51820` | WireGuard UDP port |
| `WG_DEFAULT_ADDRESS` | `10.8.0.x` | VPN subnet — clients receive addresses in this range |
| `WG_DEFAULT_DNS` | *(your local DNS IP)* | DNS server pushed to clients — set to your AdGuard Home or Pi-hole IP |
| `WG_ALLOWED_IPS` | `0.0.0.0/0` | Routes all client traffic through the VPN (full-tunnel mode) |
| `WG_PERSISTENT_KEEPALIVE` | `25` | Keepalive interval in seconds — keeps the tunnel alive through NAT |
| `UI_TRAFFIC_STATS` | `true` | Shows per-peer traffic usage in the web UI |
| `UI_CHART_TYPE` | `1` | Traffic chart style: `0`=disabled, `1`=line, `2`=area, `3`=bar |

## Linux Capabilities & Sysctls

wg-easy requires elevated privileges to manage the WireGuard network interface:

| Setting | Value | Purpose |
|---------|-------|---------|
| `cap_add: NET_ADMIN` | — | Manage network interfaces and routing |
| `cap_add: SYS_MODULE` | — | Load the WireGuard kernel module |
| `net.ipv4.ip_forward` | `1` | Enable IP forwarding so VPN clients can reach the internet |
| `net.ipv4.conf.all.src_valid_mark` | `1` | Required for WireGuard routing to function correctly |

## Setup

1. Generate the password hash on the Docker host:
   ```bash
   docker run --rm -it ghcr.io/wg-easy/wg-easy wgpw YOUR_PASSWORD
   ```
2. Place the output in `.env` — surround it with **single quotes** or escape every `$` as `$$` to prevent shell interpretation:
   ```
   PASSWORD_HASH='$2b$12$...'
   WG_HOST=vpn.yourdomain.com
   ```
3. Open UDP port `51820` on your router/firewall and forward it to the host.

## Notes

- `WG_ALLOWED_IPS=0.0.0.0/0` routes **all** client traffic through the VPN. To use split-tunnel mode (only route traffic to home network), change this to your LAN subnet (e.g. `192.168.0.0/24`).
- `WG_DEFAULT_DNS` — update this to match your local DNS server (AdGuard Home, Pi-hole, or router IP) so VPN clients benefit from local DNS filtering.
- The web UI (port `51821`) should be restricted to local network access only.
