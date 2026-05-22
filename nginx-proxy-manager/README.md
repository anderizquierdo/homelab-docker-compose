# 🔀 Nginx Proxy Manager

Nginx Proxy Manager (NPM) is the **single entry point** for all external traffic in this homelab. It acts as a reverse proxy, handling HTTPS termination and routing requests to the correct internal service — with a friendly web UI for managing proxy hosts, redirects, and SSL certificates.

## Services

### `nginx-proxy-manager`

- **Image**: [`jc21/nginx-proxy-manager:latest`](https://hub.docker.com/r/jc21/nginx-proxy-manager)
- Runs Nginx under the hood, managed via a web UI on port `81`.

### `geoipupdate`

- **Image**: [`maxmindinc/geoipupdate`](https://hub.docker.com/r/maxmindinc/geoipupdate)
- Periodically downloads and updates the MaxMind GeoLite2-Country database to `./geoip`, shared with NPM.
- Requires a free MaxMind account. Set credentials in `stack.env`.
- Updates every 48 hours (`GEOIPUPDATE_FREQUENCY=48`).

## Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| `80` | TCP | HTTP — redirects to HTTPS for all proxy hosts |
| `81` | TCP | NPM admin web UI |
| `443` | TCP | HTTPS — all proxied services |

## Volumes

| Host path | Container path | Purpose |
|-----------|---------------|---------|
| `./data` | `/data` | NPM database, proxy host configs, and SSL certificates |
| `./letsencrypt` | `/etc/letsencrypt` | Let's Encrypt account data and issued certificates |
| `./geoip` | `/data/geoip` *(NPM)* / `/usr/share/GeoIP` *(geoipupdate)* | Shared GeoIP database directory |
| `./nginx/custom` | `/data/nginx/custom` | Custom Nginx config snippets (GeoIP module, maps, per-host rules) |

## Networks

NPM is attached to **every external network** used by other stacks, enabling it to route traffic to any service container without exposing those containers directly to the host.

| Network | Purpose |
|---------|---------|
| `vaultwarden` *(external)* | Reach the Vaultwarden password manager |
| `arr-suite` *(external)* | Reach Sonarr, Radarr, Prowlarr, and Seerr |
| `p2p` *(external)* | Reach aMule and qBittorrent web UIs |
| `fileshare` *(external)* | Reach file sharing services |
| `immich` *(external)* | Reach the Immich photo library |
| `frontend` *(external)* | Reach general frontend/web services |

> To add a new service to NPM, attach its container to the appropriate shared network and create a proxy host in the NPM UI pointing to the container name and internal port.

## Environment variables

Stored in `stack.env` (not committed — create it from the table below):

| Variable | Purpose |
|----------|---------|
| `GEOIPUPDATE_ACCOUNT_ID` | MaxMind account ID |
| `GEOIPUPDATE_LICENSE_KEY` | MaxMind license key |

## GeoIP configuration

GeoIP-based filtering is implemented across three custom Nginx config files mounted at `./nginx/custom` → `/data/nginx/custom` inside the container.

### `root_top.conf` — load the GeoIP2 modules

Included at the very top of `nginx.conf`, outside any block. `load_module` directives must live here.

```nginx
load_module /usr/lib/nginx/modules/ngx_http_geoip2_module.so;
load_module /usr/lib/nginx/modules/ngx_stream_geoip2_module.so;
```

### `http_top.conf` — database, variables, and access maps

Included inside the `http` block. Defines the country variable, the allowed-country map, and a private-network bypass.

```nginx
geoip2 /data/geoip/GeoLite2-Country.mmdb {
    auto_reload 1h;
    $geoip2_data_country_code country iso_code;
}

# Allowed countries by GeoIP
map $geoip2_data_country_code $is_allowed_country {
    default 0;
    AR 1;
}

# Allowed private networks
geo $remote_addr $is_private_ip {
    default 0;
    192.168.0.0/16 1;
    172.16.0.0/12 1;
    10.0.0.0/8 1;
}
```

### `server_proxy.conf` — per-host enforcement

Included inside each proxy host's `server` block. Combines both conditions and rejects any request that matches neither.

```nginx
# Not allowed by default
set $allowed 0;

# Allow if the country is allowed
if ($is_allowed_country) {
    set $allowed 1;
}
# Allow if the request comes from a private network (internal call)
if ($is_private_ip) {
    set $allowed 1;
}

# Reject in any other case
if ($allowed = 0) {
    return 403;
}
```

> `server_proxy.conf` is applied globally to every proxy host. To exclude a specific host from GeoIP filtering, remove the include or override `$allowed` in that host's advanced config in the NPM UI.

## Notes

- The NPM admin UI (port `81`) should **not** be exposed to the internet. Access it only from the local network.
- Let's Encrypt certificates are issued and renewed automatically through NPM — no manual `certbot` commands needed.
- NPM is the **only** container in this homelab that publishes ports `80` and `443` to the host. All other services communicate with NPM over internal Docker networks without any published ports.
- Back up `./data`, `./letsencrypt`, `./geoip`, and `./nginx` to preserve your proxy configuration and avoid certificate rate-limit issues when recreating the container.
