# 📸 Immich

Immich is a self-hosted photo and video library with automatic backup from mobile devices, face recognition, object detection, and a web UI that mirrors the Google Photos experience.

## Services

### `immich-server`

- **Image**: [`ghcr.io/immich-app/immich-server`](https://github.com/immich-app/immich/pkgs/container/immich-server)
- Main application: web UI, API, background jobs (thumbnails, metadata extraction, transcoding).
- Exposed on port `2283` to the `immich` network (reachable via NPM, not published to the host).
- Has access to `/dev/dri` for hardware-accelerated video transcoding via QSV.

### `immich-machine-learning`

- **Image**: [`ghcr.io/immich-app/immich-machine-learning`](https://github.com/immich-app/immich/pkgs/container/immich-machine-learning) — **`-openvino` variant**
- Runs AI inference for face recognition and CLIP smart search.
- The OpenVINO variant offloads inference to the Intel iGPU via `/dev/dri`, reducing CPU load.

### `redis`

- **Image**: `docker.io/valkey/valkey:9` (pinned digest)
- Job queue and cache layer between `immich-server` and `immich-machine-learning`.

### `database`

- **Image**: `ghcr.io/immich-app/postgres:14-vectorchord-pgvectors` (pinned digest)
- PostgreSQL 14 with the `pgvecto.rs` and `VectorChord` extensions required for CLIP vector search.
- `DB_STORAGE_TYPE: 'HDD'` is set — remove or change to `SSD` if the database volume is on an SSD.

## Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| `2283` | TCP | Immich web UI and API (exposed to `immich` network only, proxied by NPM) |

## Volumes

| Host path | Container path | Service | Purpose |
|-----------|---------------|---------|---------|
| `${UPLOAD_LOCATION}` | `/data` | immich-server | Photo and video library |
| `/etc/localtime` | `/etc/localtime` *(ro)* | immich-server | Host timezone sync |
| `model-cache` *(named)* | `/cache` | immich-machine-learning | Downloaded AI model cache |
| `${DB_DATA_LOCATION}` | `/var/lib/postgresql/data` | database | PostgreSQL data directory |

## Networks

| Network | Purpose |
|---------|---------|
| `default` *(internal)* | Communication between stack services |
| `immich` *(external)* | Exposes `immich-server` to NPM for reverse proxying |

## Environment variables

Stored in `stack.env` (not committed — create it from the table below):

| Variable | Default | Purpose |
|----------|---------|---------|
| `UPLOAD_LOCATION` | — | Absolute host path for the photo/video library |
| `DB_DATA_LOCATION` | — | Absolute host path for PostgreSQL data |
| `TZ` | `Etc/UTC` | Timezone for the server container |
| `IMMICH_VERSION` | `v2` | Image tag — pin to a specific release for stability |
| `DB_PASSWORD` | — | PostgreSQL password |
| `DB_USERNAME` | `postgres` | PostgreSQL username |
| `DB_DATABASE_NAME` | `immich` | PostgreSQL database name |

## Hardware acceleration — QSV video transcoding

Both `immich-server` and `immich-machine-learning` have the `/dev/dri` device passed through, enabling Intel Quick Sync Video (QSV) for transcoding and OpenVINO for AI inference. This configuration targets **Intel CPUs with integrated graphics**, specifically the **Intel N150** (Alder Lake-N, Intel UHD Graphics).

### Enable QSV transcoding in the UI

1. Open the Immich web UI and go to **Administration → Video Transcoding Settings**.
2. Set **Hardware Acceleration** to `QSV`.
3. Save. Immich will use the iGPU for all subsequent transcoding jobs.

> Verify the device is accessible on the host with `ls /dev/dri` — you should see `card0` and `renderD128`. If the host user running Docker is not in the `render` or `video` group, add it: `sudo usermod -aG render,video $USER`.

## Notes

- Back up `${UPLOAD_LOCATION}` and `${DB_DATA_LOCATION}` regularly. The named `model-cache` volume does not need backing up — models are re-downloaded automatically.
- Pinned image digests for `redis` and `database` ensure reproducible deployments. Update them deliberately when upgrading.
- The Immich project does not guarantee API or database stability across minor versions. Always read the release notes before upgrading `IMMICH_VERSION`.
