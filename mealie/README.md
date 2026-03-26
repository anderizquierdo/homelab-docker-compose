# 🍽️ Mealie

Mealie is a self-hosted **recipe manager and meal planner** with a clean web interface. It supports importing recipes from URLs, tagging, meal planning, and shopping list generation.

## Services

### `mealie`

- **Image**: [`ghcr.io/mealie-recipes/mealie:latest`](https://ghcr.io/mealie-recipes/mealie)
- All-in-one container that bundles the API backend and the Vue.js frontend.

## Ports

| Port (host) | Port (container) | Purpose |
|-------------|-----------------|---------|
| `9925` | `9000` | Mealie web UI and API |

## Volumes

| Volume | Purpose |
|--------|---------|
| `mealie-data` *(named)* | Persistent application data: SQLite database, uploaded images, and backups |

## Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| `ALLOW_SIGNUP` | `false` | Disables public self-registration; new users must be created by an admin |
| `PUID` / `PGID` | `1000` / `1000` | User/group IDs for file permissions |
| `TZ` | `Europe/Madrid` | Timezone |
| `BASE_URL` | *(from `.env`)* | The public URL of the instance (e.g. `https://mealie.yourdomain.com`) |

## Resource Limits

| Resource | Limit |
|----------|-------|
| Memory | `1000 MB` |

A hard memory cap is set to prevent Mealie from consuming unbounded RAM on the host, which is relevant given it runs background tasks (recipe scraping, image processing).

## Notes

- Set `BASE_URL` in a `.env` file before starting the container; it is used for generating correct links in exports and emails.
- `ALLOW_SIGNUP: false` means the first admin account must be created through the initial setup wizard before locking down registration.
- Data is stored in a named Docker volume (`mealie-data`), which persists across container updates. Back up this volume regularly.
