# 📥 aMule

aMule is an open-source peer-to-peer file sharing client for the **eDonkey (ed2k) and Kademlia (KAD)** networks. It is a fully featured alternative to eMule, accessible via a web UI or remote GUI.

## Services

### `amule`

- **Image**: [`ngosang/amule`](https://hub.docker.com/r/ngosang/amule)
- A long-running aMule daemon with built-in web UI and remote GUI support.

## Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| `4662` | TCP | ed2k client-to-server and client-to-client transfers |
| `4665` | UDP | ed2k global search (TCP port + 3) |
| `4672` | UDP | ed2k UDP extension |
| `4711` | TCP | Web UI *(commented out — exposed via reverse proxy)* |
| `4712` | TCP | Remote GUI / webserver / cmd *(commented out)* |

> The web UI and remote GUI ports are not published to the host. They are intended to be accessed through Nginx Proxy Manager over the internal `p2p` Docker network.

## Volumes

| Path (host) | Path (container) | Purpose |
|-------------|-----------------|---------|
| `./config` | `/home/amule/.aMule` | aMule configuration files |
| `/mnt/p2p/amule/incoming` | `/incoming` | Completed downloads |
| `/mnt/p2p/amule/temp` | `/temp` | In-progress / partial downloads |

## Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| `PUID` / `PGID` | `1000` / `2000` | User/group IDs for file permissions |
| `UMASK` | `002` | File creation mask |
| `TZ` | `Europe/Madrid` | Timezone |
| `GUI_PWD` | *(from `.env`)* | Password for the remote GUI |
| `WEBUI_PWD` | *(from `.env`)* | Password for the web UI |
| `MOD_AUTO_RESTART_ENABLED` | `true` | Enables automatic scheduled restart |
| `MOD_AUTO_RESTART_CRON` | `0 6 * * *` | Restarts daily at 06:00 |
| `MOD_AUTO_SHARE_ENABLED` | `false` | Auto-sharing of directories is disabled |
| `MOD_FIX_KAD_GRAPH_ENABLED` | `true` | Fixes the KAD graph display |
| `MOD_FIX_KAD_BOOTSTRAP_ENABLED` | `true` | Fixes KAD network bootstrapping |

## Networks

| Network | Purpose |
|---------|---------|
| `p2p` *(external)* | Shared network for P2P services; gives Nginx Proxy Manager access to the web UI |

## Notes

- Set `GUI_PWD` and `WEBUI_PWD` in a `.env` file before starting the container.
- The container restarts automatically every day at 06:00 to keep connections fresh.
- `PGID=2000` is intentionally different from `PUID=1000` — ensure the target data directories grant write access to group `2000`.
