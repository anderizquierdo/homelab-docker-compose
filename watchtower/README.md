# 🗼 Watchtower

Watchtower automatically **updates running Docker containers** whenever a newer image is available. It pulls the latest image, gracefully stops the old container, and starts a new one with the same configuration — no manual intervention needed. Update results are reported via **Telegram**.

## Services

### `watchtower`

- **Image**: [`nickfedor/watchtower`](https://hub.docker.com/r/nickfedor/watchtower)
- Monitors all running containers on the host and updates them on schedule.

## Volumes

| Host path | Container path | Purpose |
|-----------|---------------|---------|
| `/var/run/docker.sock` | `/var/run/docker.sock` | Docker socket access — required for Watchtower to manage containers |

> ⚠️ Mounting the Docker socket grants the container root-level access to the Docker daemon. Watchtower requires this to inspect and restart containers.

## Schedule

```
--schedule "0 0 4 * * *"
```

Watchtower runs once every night at **04:00**. It uses a 6-field cron expression (seconds included): `second minute hour day month weekday`.

## Cleanup

The `--cleanup` flag is set, which removes old images after a successful update to prevent disk space accumulation.

## Telegram Notifications

Watchtower sends a Telegram message after each update run **only if at least one container was updated or failed**. Runs with no changes produce no notification.

| Variable | Description |
|----------|-------------|
| `WATCHTOWER_NOTIFICATION_REPORT` | Enables the structured report format |
| `WATCHTOWER_NOTIFICATION_URL` | Apprise-compatible Telegram URL: `telegram://<BOT_TOKEN>@telegram?chats=<CHAT_ID>` |
| `WATCHTOWER_NOTIFICATION_TEMPLATE` | Custom Go template — reports count of scanned/updated/failed containers and lists each updated image with its old and new digest |

Set `BOT_TOKEN` and `CHAT_ID` in a `.env` file. Create a Telegram bot via [@BotFather](https://t.me/BotFather) to obtain a token, and get your chat ID from [@userinfobot](https://t.me/userinfobot).

## Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| `WATCHTOWER_NOTIFICATION_REPORT` | `"true"` | Use the summary report format instead of per-event messages |
| `WATCHTOWER_NOTIFICATION_URL` | *(from `.env`)* | Telegram notification target via Apprise URL scheme |
| `WATCHTOWER_NOTIFICATION_TEMPLATE` | *(inline Go template)* | Custom message format for update reports |

## Notes

- Watchtower updates **all** containers on the host by default. To exclude a specific container, add the label `com.centurylinklabs.watchtower.enable=false` to it.
- Running updates at 04:00 minimises the chance of an update disrupting active use.
- The `nickfedor/watchtower` fork is used instead of the archived `containrrr/watchtower` original.
