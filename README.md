# dozzle

A docker compose deployment of [Dozzle](https://dozzle.dev), a real-time log
viewer for Docker containers, one instance per docker host, behind an
[nginx-proxy](https://github.com/nginx-proxy/nginx-proxy).

* reads container logs and stats via the docker socket (read-only)
* no host ports — published through the proxy at `dozzle.<BASE_DOMAIN>`
* state (user settings, notification rules, `users.yml`) persists in a named
  volume at `/data`
* all env vars live in `.env` (local, gitignored — copy from `env.example`)
* compose fails fast when a required variable is unset

## Layout

| Path | Purpose |
| --- | --- |
| `docker-compose.yml` | Service definition, nginx-proxy wiring |
| `dozzle/README.md` | Service notes: upstream image choice, auth, security |
| `env.example` | Tracked source of truth for required and optional variables |
| `.env` | Local, gitignored copy of `env.example` with real values |

## Prerequisites

* docker compose
* an existing nginx-proxy network (created via docker compose on the host
  running nginx-proxy)

## Setup

1. Copy `env.example` to `.env` and edit `.env`:

   ```sh
   cp env.example .env
   ```

   * `NGINX_PROXY_NETWORK` — name of the shared nginx-proxy network
   * `BASE_DOMAIN` — the root domain this host serves (e.g. `example.com`)
   * `DOZZLE_VERSION` — pinned upstream image tag (e.g. `v10.9.2`)
   * `TZ` — optional, default `UTC`
   * `DOZZLE_HOSTNAME` — optional, name shown in the dozzle header
   * `DOZZLE_LEVEL` — optional, log verbosity, default `info`
   * `DOZZLE_AUTH_PROVIDER` — optional, `none` (default) or `simple`
     (see `dozzle/README.md`)

2. Sanity-check and start (no build step — the upstream image IS the service):

   ```sh
   docker compose config  # fails fast on missing vars
   docker compose up -d
   docker compose ps
   ```

3. Open `http://dozzle.<BASE_DOMAIN>`. This deployment is HTTP-only in every
   variant: it declares no TLS opt-in (no `ACME_HOST`, no
   `GEN_SELF_SIGNED_CERT`), so the proxy serves it over plain HTTP whether the
   cluster is LAN-only or internet-facing.

## Operational notes

* **Upgrading**: bump `DOZZLE_VERSION` in `.env`, then
  `docker compose pull dozzle && docker compose up -d dozzle`. Pin a specific
  tag (never `latest`) so upgrades are deliberate.
* **Resetting state**: `docker compose rm -sf dozzle` +
  `docker volume rm dozzle_dozzle-data`, then `up -d` (loses users and
  settings; logs themselves are never stored by dozzle).
* **Actions / shell stay disabled**: `DOZZLE_ENABLE_ACTIONS` and
  `DOZZLE_ENABLE_SHELL` are deliberately not set — they allow starting,
  stopping, recreating, and executing commands inside containers. Enable only
  if the instance is reachable solely through your identity-aware proxy.
* **Homepage**: this service appears on the dashboard via its `homepage.*`
  labels; `homepage.href` derives from the same `x-hosts` anchor as
  `VIRTUAL_HOST`, so changing `BASE_DOMAIN` in `.env` moves the whole stack.