# dozzle service

Upstream image: `amir20/dozzle:<tag>` (Docker Hub). There is no Dockerfile in
this directory — the upstream image IS the service (COMPOSE spec rule 6), so
there is no build step. Likewise there is no custom entrypoint (spec rule 8:
entrypoints exist only for custom-built images that need first-run seeding;
this image needs none — documented here).

## Why the upstream image, and which tag

* The image is a single static Go binary on `scratch` (~20 MB compressed) —
  the whole app with no distro packages to patch. The alpine variant
  (`<tag>-alpine`, e.g. `v10.9.2-alpine`) exists for the edge case of older
  Docker engines whose API version the pinned SDK no longer negotiates; the
  upstream FAQ recommends scratch for everything else.
* The tag is pinned via `DOZZLE_VERSION` in `.env` (default `v10.9.2`, current
  stable at the time of writing). Never `latest` — upgrades should be
  deliberate. Bump the tag, pull, recreate.

## Runtime characteristics

* Runs as root (the upstream Dockerfile sets no `USER`). No
  `PUID`/`PGID` privilege drop applies (spec rule 9: only when a custom
  entrypoint exists), and the image needs the docker socket anyway, so it
  already operates at container-engine privilege level. Least privilege is
  preserved by mounting the socket read-only and granting nothing else.
* The docker socket bind mount at `/var/run/docker.sock` is the documented
  spec rule 7 special case (socket, read-only). Everything stateful lives in
  the standard named volume `dozzle-data` at `/data` — no host bind mounts
  for stateful data.
* No built-in healthcheck and none added: the scratch image has no shell, so
  `CMD-SHELL` probes are impossible and no probe binary ships in the image.

## Environment variables

| Variable | Default | Purpose |
| --- | --- | --- |
| `DOZZLE_HOSTNAME` | `dozzle` | Name shown in the header / multi-host menu |
| `DOZZLE_LEVEL` | `info` | Log verbosity: `debug` \| `info` \| `warn` \| `error` |
| `DOZZLE_AUTH_PROVIDER` | `none` | `none` or `simple` (see below) |

Deliberately NOT set: `DOZZLE_ENABLE_ACTIONS`, `DOZZLE_ENABLE_SHELL`. Both
grant control over containers (start/stop/recreate, arbitrary shell access);
they stay off unless the instance is reachable only through an
identity-aware proxy. See https://dozzle.dev/guide/actions and
https://dozzle.dev/guide/shell.

## Authentication

Default posture is LAN-only (`DOZZLE_AUTH_PROVIDER=none`), consistent with the
other projects in this workspace: access is trusted to the LAN. This
deployment is HTTP-only — it declares no TLS opt-in. Dozzle's own docs say to
put it behind authentication whenever it is reachable from the public
internet — it grants full read access to every container's logs.

To enable login with the built-in `simple` provider:

1. Create `users.yml` in this directory with a bcrypt hash — the upstream
   image ships a generator:

   ```sh
   docker run --rm -it amir20/dozzle:v10.9.2 generate admin \
     --password 'choose-a-password' --email you@example.com --name Admin
   ```

2. Mount it as `/data/users.yml` and flip the provider. A compose `secrets:`
   entry (official recipe) avoids a bind mount and keeps the file out of the
   container filesystem diff:

   ```yaml
   services:
     dozzle:
       environment:
         DOZZLE_AUTH_PROVIDER: simple
       secrets:
         - source: dozzle-users
           target: /data/users.yml
   secrets:
     dozzle-users:
       file: ./users.yml
   ```

   then `docker compose up -d dozzle` and log in with the generated user.

3. `DOZZLE_AUTH_TTL` (e.g. `48h`, duration only: `s`/`m`/`h`) optionally caps
   session lifetime — see https://dozzle.dev/guide/authentication.

Users, settings and notification rules are written to `/data`. The users file
is read on every start (`users.yml` wins over `users.yaml`); changes take
effect on recreate. Use the "Resetting state" procedure in the project
README.md to start over.