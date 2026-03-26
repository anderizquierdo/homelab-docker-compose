# 📻 Wavelog

Wavelog is a self-hosted **amateur (ham) radio logging application**. It allows radio operators to log contacts (QSOs), track awards (DXCC, SOTA, POTA, etc.), upload logs to LoTW/eQSL, and view statistics about their activity.

## Services

### `wavelog-db`

- **Image**: [`mariadb:11.3`](https://hub.docker.com/_/mariadb)
- MariaDB database backend that stores all QSO data and application state.
- Uses a randomly generated root password (`MARIADB_RANDOM_ROOT_PASSWORD: yes`) for security — the application connects via the dedicated `MARIADB_USER` account instead.

### `wavelog-main`

- **Image**: [`ghcr.io/wavelog/wavelog:latest`](https://ghcr.io/wavelog/wavelog)
- The Wavelog PHP application served over Apache. Depends on `wavelog-db` and will start only after the database container is up.
- **Internal port**: `80` (mapped to host port `8086`)

## Ports

| Port (host) | Port (container) | Purpose |
|-------------|-----------------|---------|
| `8086` | `80` | Wavelog web UI |

## Volumes

| Volume | Used by | Purpose |
|--------|---------|---------|
| `wavelog-dbdata` *(named)* | `wavelog-db` | MariaDB data directory — all QSO data |
| `wavelog-config` *(named)* | `wavelog-main` | Wavelog Docker-specific configuration files |
| `wavelog-uploads` *(named)* | `wavelog-main` | User-uploaded files (e.g. images) |
| `wavelog-userdata` *(named)* | `wavelog-main` | User data (logs, exports) |

## Environment Variables

### `wavelog-db`

| Variable | Description |
|----------|-------------|
| `MARIADB_RANDOM_ROOT_PASSWORD` | Generates a secure random root password at first startup |
| `MARIADB_DATABASE` | Database name *(from `.env`)* |
| `MARIADB_USER` | Application database user *(from `.env`)* |
| `MARIADB_PASSWORD` | Application database password *(from `.env`)* |

### `wavelog-main`

| Variable | Value | Description |
|----------|-------|-------------|
| `CI_ENV` | `docker` | Tells Wavelog it is running in a Docker environment, applying Docker-specific configuration |

## Setup

Set the following variables in a `.env` file before starting the stack:

```
MARIADB_DATABASE=wavelog
MARIADB_USER=wavelog
MARIADB_PASSWORD=your_secure_password
```

## Notes

- The database credentials in `.env` must be consistent between `wavelog-db` and `wavelog-main` — Wavelog reads them at startup to connect to MariaDB.
- All four named volumes should be backed up regularly to protect QSO history.
- `wavelog-main` uses `depends_on: wavelog-db`, but this only waits for the container to start, not for MariaDB to be fully ready. On the very first run, Wavelog may need a moment for the DB to initialise before the setup wizard completes successfully.
