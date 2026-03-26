# 🔀 Nginx Proxy Manager

Nginx Proxy Manager (NPM) is the **single entry point** for all external traffic in this homelab. It acts as a reverse proxy, handling HTTPS termination and routing requests to the correct internal service — with a friendly web UI for managing proxy hosts, redirects, and SSL certificates.

## Services

### `app`

- **Image**: [`jc21/nginx-proxy-manager:latest`](https://hub.docker.com/r/jc21/nginx-proxy-manager)
- Runs Nginx under the hood, managed via a web UI on port `81`.

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

## Networks

NPM is attached to **every external network** used by other stacks, enabling it to route traffic to any service container without exposing those containers directly to the host.

| Network | Purpose |
|---------|---------|
| `vaultwarden` *(external)* | Reach the Vaultwarden password manager |
| `arr-suite` *(external)* | Reach Sonarr, Radarr, Prowlarr, and Seerr |
| `p2p` *(external)* | Reach aMule and qBittorrent web UIs |

> To add a new service to NPM, attach its container to the appropriate shared network and create a proxy host in the NPM UI pointing to the container name and internal port.

## Notes

- The NPM admin UI (port `81`) should **not** be exposed to the internet. Access it only from the local network.
- Let's Encrypt certificates are issued and renewed automatically through NPM — no manual `certbot` commands needed.
- NPM is the **only** container in this homelab that publishes ports `80` and `443` to the host. All other services communicate with NPM over internal Docker networks without any published ports.
- Back up both `./data` and `./letsencrypt` to preserve your proxy configuration and avoid certificate rate-limit issues when recreating the container.
