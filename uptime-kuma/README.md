# 📡 Uptime Kuma

Uptime Kuma is a self-hosted **monitoring and status page** tool. It checks the availability of services (HTTP, TCP, DNS, ping, and more) and sends notifications when they go down or come back up.

## Services

### `uptime-kuma`

- **Image**: [`louislam/uptime-kuma:1`](https://hub.docker.com/r/louislam/uptime-kuma)
- A single all-in-one container with a real-time web UI powered by WebSockets.

## Ports

| Port (host) | Port (container) | Purpose |
|-------------|-----------------|---------|
| `3001` | `3001` | Uptime Kuma web UI |

## Volumes

| Host path | Container path | Purpose |
|-----------|---------------|---------|
| `./data` | `/app/data` | SQLite database, configuration, and status history |

## Notes

- The image is pinned to the major version tag (`1`) rather than `latest` to avoid unexpected breaking changes from a future v2 release.
- On first start, navigate to `http://<host>:3001` to create the admin account. There is no default credential.
- Uptime Kuma can monitor services inside Docker networks by using container names as hostnames — add it to the relevant Docker networks if internal monitoring is needed.
- The `./data` directory contains the SQLite database with all monitor history; back it up regularly to preserve uptime records and alert configurations.
