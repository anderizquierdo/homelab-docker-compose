# 🎬 Arr Suite

A collection of media automation tools that work together to find, download, and manage your media library. The stack integrates with download clients (e.g. qBittorrent, aMule) on the shared `p2p` network.

## Services

### `sonarr`

- **Image**: [`ghcr.io/hotio/sonarr`](https://ghcr.io/hotio/sonarr)
- TV series management: monitors RSS feeds for new episodes, sends them to a download client, and organises the library.
- **Internal port**: `8989`

### `radarr`

- **Image**: [`ghcr.io/hotio/radarr`](https://ghcr.io/hotio/radarr)
- Movie management: same workflow as Sonarr but for films.
- **Internal port**: `7878`

### `prowlarr`

- **Image**: [`ghcr.io/hotio/prowlarr`](https://ghcr.io/hotio/prowlarr)
- Indexer manager/proxy: manages torrent and Usenet indexers in one place and syncs them to Sonarr and Radarr automatically, eliminating manual configuration in each app.
- **Internal port**: `9696`

### `seerr`

- **Image**: [`ghcr.io/seerr-team/seerr`](https://ghcr.io/seerr-team/seerr)
- Media request and discovery front-end: allows users to request movies and TV shows, which are automatically forwarded to Radarr/Sonarr.
- **Internal port**: `5055`

## Ports

No ports are published to the host. All services are accessed exclusively through **Nginx Proxy Manager** over the internal `arr-suite` Docker network.

## Volumes

| Service | Host path | Container path | Purpose |
|---------|-----------|---------------|---------|
| `sonarr` | `./sonarr` | `/config` | Sonarr configuration and database |
| `sonarr` | `/mnt/p2p` | `/data` | Shared P2P data root (downloads + media library) |
| `radarr` | `./radarr` | `/config` | Radarr configuration and database |
| `radarr` | `/mnt/p2p` | `/data` | Shared P2P data root (downloads + media library) |
| `prowlarr` | `./prowlarr` | `/config` | Prowlarr configuration and database |
| `seerr` | `./seerr` | `/app/config` | Seerr configuration |

> Sonarr and Radarr both mount `/mnt/p2p` as `/data`. Using a single shared mount allows hardlinks between the downloads folder and the media library, avoiding unnecessary disk duplication.

## Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| `PUID` / `PGID` | `1000` / `2000` | User/group IDs for file permissions (Sonarr & Radarr) |
| `PUID` / `PGID` | `1000` / `1000` | User/group IDs for Prowlarr |
| `UMASK` | `002` | File creation mask |
| `TZ` | `Europe/Madrid` | Timezone |

## Networks

| Network | Attached services | Purpose |
|---------|-------------------|---------|
| `arr-suite` *(external)* | All four services | Internal communication between arr apps and access via Nginx Proxy Manager |
| `p2p` *(external)* | Sonarr, Radarr | Access to download clients (qBittorrent, aMule) on the P2P network |

> `prowlarr` and `seerr` are on `arr-suite` only — they do not need direct access to download clients.

## Notes

- Configure Prowlarr first, then link it to Sonarr and Radarr via their respective API keys.
- The `seerr` container uses `init: true` to properly handle signal forwarding and zombie process reaping.
- `PGID=2000` for Sonarr and Radarr must match the group that owns the files under `/mnt/p2p`.
