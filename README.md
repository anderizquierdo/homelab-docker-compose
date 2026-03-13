# 🏠 Homelab Docker Composes

A collection of Docker Compose stacks powering my homelab. Each directory contains a self-contained stack for a specific service or set of related services.

---

## 📐 Network Architecture

### Nginx Proxy Manager as the single entry point

All external traffic is routed through **[Nginx Proxy Manager](https://nginxproxymanager.com/)** (NPM), which acts as the sole reverse proxy and entry point for every service. This provides:

- **Centralized routing** — one place to manage all proxy hosts and redirects.
- **Security out of the box** — HTTPS for every service.
- **Automatic, free certificates** — Let's Encrypt integration handles issuance and renewal automatically.

### Per-stack shared networks (no exposed ports)

Services do **not** expose ports directly to the host. Instead, each stack defines a dedicated Docker network that is shared with NPM, allowing NPM to reach the service containers over the Docker internal network.

```
Internet
   │
   ▼
[Nginx Proxy Manager]
   │         │         │
   │ net_a   │ net_b   │ net_c   ...
   ▼         ▼         ▼
[Stack A] [Stack B] [Stack C]
```

If a stack contains services that do **not** need to be proxied (e.g. a database backend that only talks to its own app), those services are placed on an **internal-only network** isolated from NPM. In that case, a typical stack will have two networks:

| Network | Purpose |
|---|---|
| `default` | Communication between containers within the same stack only |
| `<stack_name>` | Shared with NPM; only the service that NPM needs to reach is attached |

### Why this reduces the attack surface

- **No host port exposure** — there are no published ports (e.g. `ports: - "8080:8080"`) anywhere in the stacks. A port scan of the host reveals nothing but NPM's 80/443.
- **Network segmentation** — each stack lives in its own Docker network. A compromised container in Stack A cannot reach containers in Stack B because they share no network.
- **Least-privilege connectivity** — NPM is attached to many networks simultaneously, but each individual service only sees its own stack's network (and optionally NPM). No service has unnecessary visibility into other stacks.
- **Internal services stay internal** — database containers, caches, and other backend services that never need external access are placed exclusively on `*_internal` networks with `internal: true`, making them unreachable from outside the stack by design.

---

## ⚙️ Docker Daemon Configuration

`/etc/docker/daemon.json`:

```json
{
  "registry-mirrors": ["https://mirror.gcr.io"],
  "default-address-pools": [
    {"base": "172.16.0.0/12", "size": 24}
  ]
}
```

### `registry-mirrors` — Google Container Registry mirror

Docker Hub is intermittently blocked by Spanish ISPs during football matches due to automated traffic blocks issued by *La Liga*. Using Google's public Docker Hub mirror (`mirror.gcr.io`) routes pull requests through Google's infrastructure, bypassing these blocks transparently with no changes required in any Compose file.

### `default-address-pools` — Custom subnet allocation

Docker's default address pool is:

```json
{
  "default-address-pools": [
    {"base":"172.17.0.0/16","size":16},
    {"base":"172.18.0.0/16","size":16},
    {"base":"172.19.0.0/16","size":16},
    {"base":"172.20.0.0/14","size":16},
    {"base":"172.24.0.0/14","size":16},
    {"base":"172.28.0.0/14","size":16},
    {"base":"192.168.0.0/16","size":20}
  ]
}
```

Each entry allocates subnets of size `/16` (65,536 IPs) from the given base range, except the last one which allocates `/20` blocks (4,096 IPs). The total number of networks available across all pools is only 31 — and several of those pools sit in the `192.168.0.0/16` range. This default is optimised for a small number of large stacks — not a homelab with many small, isolated stacks.

The custom configuration uses:

```json
{
  "default-address-pools": [
    {"base": "172.16.0.0/12", "size": 24}
  ]
}
```

The `172.16.0.0/12` divided into `/24` blocks provides 4,096 networks with 256 addresses each, much more suitable for a homelab with many stacks of 2-5 containers each.

**Why not use the defaults?**

1. **VLAN conflicts** — The default pool includes `192.168.0.0/16`, which overlaps with some of my LAN subnets. In a segmented network with multiple VLANs in 192.168.x.y segment (main, IoT, VPN, Guest), Docker would create routes that conflict with real LAN traffic, causing routing failures or unexpected connectivity between the host and the LAN. Moving to `172.16.0.0/12` eliminates this entirely.

2. **Many small stacks** — The default pool is dimensioned for fewer, larger networks. With one isolated network per stack (plus internal networks), the default pool exhausts quickly. The custom pool provides 4,096 `/24` networks, each still offering 254 usable IPs — far more than any individual stack needs, and enough to scale the homelab without ever having to reconfigure.

---
## 📝 License

Personal homelab configuration — feel free to use as inspiration.
