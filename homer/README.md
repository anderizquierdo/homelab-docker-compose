# 🏠 Homer

Homer is a dead-simple, static **homelab dashboard** served as a single-page web app. It provides a central landing page with links to all your self-hosted services.

## Services

### `app`

- **Image**: [`b4bz/homer`](https://hub.docker.com/r/b4bz/homer)
- Serves a fully static HTML/JS dashboard; no database, no backend logic.

## Ports

| Port (host) | Port (container) | Purpose |
|-------------|-----------------|---------|
| `8080` | `8080` | Homer web UI |

> Unlike most other services in this repo, Homer publishes its port directly to the host. This makes the dashboard accessible on the local network without going through Nginx Proxy Manager.

## Volumes

| Host path | Container path | Purpose |
|-----------|---------------|---------|
| `./assets` | `/www/assets` | Dashboard configuration (`config.yml`) and static assets (icons, etc.) |

## Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| `INIT_ASSETS` | `1` | Copies default assets into the volume on first start if it is empty |
| `IPV6_DISABLE` | `1` | Disables IPv6 on the server to avoid binding issues |

## User

The container runs as user `1000:1000`, matching the host user that owns `./assets`, to avoid permission issues when editing the config file directly on the host.

## Configuration

Edit `./assets/config.yml` to customise the dashboard — add service tiles, groups, links, icons, and theme settings. Changes are picked up immediately on page refresh; no container restart needed.

## Notes

- Homer is a purely static site — there is no authentication. Treat it as an internal-only convenience tool.
- The `config.yml` file is the single source of truth for the dashboard layout. Back it up alongside the rest of your homelab configuration.
