# 🔐 Vaultwarden

Vaultwarden is a lightweight, self-hosted **Bitwarden-compatible password manager** server. It is fully compatible with all official Bitwarden clients (browser extensions, mobile apps, desktop apps). This stack also includes an automatic hourly backup service.

## Services

### `vaultwarden`

- **Image**: [`vaultwarden/server:latest`](https://hub.docker.com/r/vaultwarden/server)
- Implements the Bitwarden API with a much smaller resource footprint than the official server.

### `vaultwarden-backup`

- **Image**: [`bruceforce/vaultwarden-backup`](https://hub.docker.com/r/bruceforce/vaultwarden-backup)
- Runs alongside Vaultwarden and automatically backs up the data volume on a schedule.
- Uses `init: true` for proper signal handling.
- Runs with `network_mode: none` — it has no network access whatsoever, as it only needs to access local volumes.

## Ports

No ports are published to the host. Vaultwarden is accessed exclusively through **Nginx Proxy Manager** over the internal `vaultwarden` Docker network.

## Volumes

| Volume | Used by | Purpose |
|--------|---------|---------|
| `vaultwarden` *(named)* | `vaultwarden`, `vaultwarden-backup` | Vaultwarden data: SQLite database, attachments, and configuration |
| `backup` *(named)* | `vaultwarden-backup` | Backup destination |

Both services share the `vaultwarden` volume so the backup container can read the live data directly.

## Environment Variables

### `vaultwarden`

| Variable | Value | Description |
|----------|-------|-------------|
| `DOMAIN` | *(from `.env`)* | The public HTTPS URL (e.g. `https://vault.yourdomain.com`). Required for Web Vault and Duo 2FA |
| `SIGNUPS_ALLOWED` | `"true"` | Controls whether new users can self-register |

> ⚠️ Set `SIGNUPS_ALLOWED` to `"false"` after creating your account to prevent strangers from registering on your instance.

### `vaultwarden-backup`

| Variable | Value | Description |
|----------|-------|-------------|
| `CRON_TIME` | `0 * * * *` | Backup runs every hour at minute 0 |
| `BACKUP_USE_DEDUPE` | `true` | Enables deduplication to reduce backup storage consumption |
| `TZ` | `Europe/Madrid` | Timezone for cron scheduling |

## Networks

| Network | Purpose |
|---------|---------|
| `vaultwarden` *(external)* | Allows Nginx Proxy Manager to reach the Vaultwarden container |

`vaultwarden-backup` uses `network_mode: none` and is completely isolated from all networks.

## Notes

- Set `DOMAIN` in a `.env` file. The domain must match the URL used by clients, or certain features (Web Vault, 2FA) will not work correctly.
- Backups are written to the `backup` named volume. Mount it or copy it to an external location for off-host backups.
- Vaultwarden is a security-critical service — keep the image updated and ensure it is only reachable over HTTPS.
