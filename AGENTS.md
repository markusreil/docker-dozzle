# AGENTS.md

Ongoing rules for working in this repo. This is a summary — see
[README.md](README.md) and [dozzle/README.md](dozzle/README.md) for detail, and
`COMPOSE-SPEC.md` for the full spec these rules come from.

## Configuration

- All deployment configuration lives in `.env` (hidden, gitignored). Track
  `env.example` instead; keep both in the same order and shape with a comment
  per variable, and copy with `cp env.example .env` before starting.
- Required vars use `${VAR:?...}` so `docker compose config` fails fast. Never
  replace a required var with a silent `${VAR:-default}`.
- `env.example` / `.env` are grouped: global vars at the top, then one
  `# --- dozzle ---` section.

## Compose

- One compose file; project name set once (`name: dozzle`). Never use
  `container_name:` — rely on default container naming.
- No host `ports:` mappings. The service joins the external `nginx-proxy`
  network (`${NGINX_PROXY_NETWORK:?...}`, `external: true`) and must start
  after the proxy cluster.
- Hostnames are defined once as `x-hosts` anchors and referenced everywhere
  (`VIRTUAL_HOST`, Homepage `homepage.href`). Change `BASE_DOMAIN` in `.env` to
  move the whole stack.
- `restart: unless-stopped` by default.
- Stateful data uses named volumes (`dozzle-data`); no host bind mounts for
  state. The read-only docker socket is the documented exception.
- Homepage integration via `homepage.*` labels.

## Proxy contract

- The service declares **both** TLS opt-ins — `ACME_HOST` (real certificate,
  honoured by the internet-facing variant) and `GEN_SELF_SIGNED_CERT`
  (self-signed, honoured by the LAN variant) — so the proxy variant decides
  which applies. Never declare only one: a lone opt-in pins the service to a
  single variant.
- `GEN_SELF_SIGNED_CERT` is driven from `.env` via
  `DOZZLE_GEN_SELF_SIGNED_CERT` (default `false`). With it `false` a LAN
  cluster serves plain HTTP; set it `true` for full variant-agnostic TLS.
- Use the `ACME_*` spelling for every ACME variable; the `LETSENCRYPT_*` aliases
  are deprecated and must not be used.

## Service image

- The upstream image *is* the service (`amir20/dozzle`, scratch base). No
  Dockerfile, no custom entrypoint, no `PUID`/`PGID` drop; the reasoning is in
  `dozzle/README.md`.

## Changelog

- Keep `CHANGELOG.md` in
  [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format: fill the
  `## [Unreleased]` subsections as changes are made. Do not create release
  sections unless asked.

## Verify

```sh
cp env.example .env
docker compose config   # must pass; fails fast on missing vars
docker compose up -d
docker compose ps
```
