# 🔄 Syncthing

Syncthing is a **continuous, decentralised file synchronisation** tool. It syncs files between devices directly (peer-to-peer), without relying on any cloud service or central server.

## Services

### `app`

- **Image**: [`syncthing/syncthing`](https://hub.docker.com/r/syncthing/syncthing) (official image)
- Runs the Syncthing daemon and its built-in web UI.

## Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| `8384` | TCP | Syncthing web UI |
| `22000` | TCP | File transfer (TCP) |
| `22000` | UDP | File transfer (QUIC) |
| `21027` | UDP | Local device discovery broadcasts |

## Volumes

| Host path | Container path | Purpose |
|-----------|---------------|---------|
| `/mnt/syncthing` | `/var/syncthing` | All Syncthing data: configuration, index database, and synced folders |

## Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| `PUID` / `PGID` | `1010` / `1010` | User/group IDs for file permissions |

> The dedicated UID/GID `1010` (different from the default `1000`) is used to isolate Syncthing's file ownership from other services.

## Health Check

The container includes a built-in health check that polls the Syncthing REST API every minute:

```
curl -fkLsS -m 2 127.0.0.1:8384/rest/noauth/health | grep -o OK
```

| Parameter | Value |
|-----------|-------|
| Interval | 1 minute |
| Timeout | 10 seconds |
| Retries | 3 |

## Notes

- All synced folders must be located under `/mnt/syncthing` (mapped to `/var/syncthing` in the container) so that Syncthing can read and write them with the correct permissions.
- The web UI (port `8384`) is published directly to the host for local network access. Consider restricting it to the local network via firewall rules.
- File transfer ports (`22000`) must be reachable from remote devices — open them in your router/firewall if you need to sync with devices outside the local network.
- Port `21027/udp` is only needed for local network discovery; it is not required for remote peers.
