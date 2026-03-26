# 🌊 qBittorrent

qBittorrent is an open-source **BitTorrent client** with a feature-rich web UI. This setup uses the LinuxServer.io image and replaces the default web UI with **VueTorrent**, a modern Vue.js-based alternative.

## Services

### `qbittorrent`

- **Image**: [`lscr.io/linuxserver/qbittorrent:latest`](https://hub.docker.com/r/linuxserver/qbittorrent)
- Runs the qBittorrent daemon with a web UI.
- **VueTorrent mod**: [`ghcr.io/vuetorrent/vuetorrent-lsio-mod:latest`](https://ghcr.io/vuetorrent/vuetorrent-lsio-mod) replaces the built-in web UI with the VueTorrent interface automatically via `DOCKER_MODS`.

## Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| `8080` | TCP | Web UI *(commented out — exposed via reverse proxy)* |
| `6881` | TCP | BitTorrent peer connections |
| `6881` | UDP | BitTorrent peer connections (UDP) |

> The web UI port (`8080`) is not published to the host. It is accessed through **Nginx Proxy Manager** over the internal `p2p` Docker network.

## Volumes

| Host path | Container path | Purpose |
|-----------|---------------|---------|
| `./config` | `/config` | qBittorrent configuration, session data, and resume files |
| `/mnt/p2p/torrent` | `/downloads` | Download destination directory |

## Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| `PUID` / `PGID` | `1000` / `2000` | User/group IDs for file permissions |
| `UMASK` | `002` | File creation mask |
| `TZ` | `Europe/Madrid` | Timezone |
| `WEBUI_PORT` | `8080` | Web UI port (must match the internal port used by Sonarr/Radarr) |
| `TORRENTING_PORT` | `6881` | Port used for peer connections |
| `DOCKER_MODS` | `ghcr.io/vuetorrent/vuetorrent-lsio-mod:latest` | Installs the VueTorrent UI mod on container start |

## Networks

| Network | Purpose |
|---------|---------|
| `p2p` *(external)* | Shared network for P2P services; gives Nginx Proxy Manager and the arr suite access to qBittorrent |

## Notes

- `PGID=2000` must match the group that owns `/mnt/p2p/torrent` so downloaded files are writable by Sonarr and Radarr.
- VueTorrent is downloaded and installed automatically by the LSIO mod system on each container start. Ensure the container has internet access during startup.
- Configure qBittorrent as a download client in Sonarr and Radarr using its container name (`qbittorrent`) and port `8080` over the shared `p2p` network.
