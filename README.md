# Printer farm infrastructure

Public-safe infrastructure repository for the 3D printing farm homelab.

## For humans and agents

| Read first | Why |
|------------|-----|
| [`AGENT_HANDOFF.md`](AGENT_HANDOFF.md) | Continuity: live paths vs repo, Klipper/USB, Git remote, done vs next |
| [`AGENTS.md`](AGENTS.md) | Short checklist for Cursor / automation |
| [`docs/DIRECTORY_LAYOUT.md`](docs/DIRECTORY_LAYOUT.md) | Where folders live on the server |

## Repository contents

* [`VISION.MD`](VISION.MD) — long-term architecture and roadmap
* [`INFRASTRUCTURE.MD`](INFRASTRUCTURE.MD) — deployed stack, ports, backups, **live host paths** vs clone paths
* [`docker/docker-compose.yml`](docker/docker-compose.yml) — base stack (keep in sync with production when you change it)
* Sanitized Home Assistant and ESPHome templates under `docker/`
* `secrets.yaml.example` files — copy to real `secrets.yaml` locally; never commit secrets

## Services in this stack

* Home Assistant
* ESPHome
* Uptime Kuma
* Portainer

## Excluded from Git

Runtime state, databases, logs, real `secrets.yaml`, Home Assistant `.storage/`, ESPHome `.esphome/` build cache — see [`.gitignore`](.gitignore).

## Clone setup

1. Copy example secrets:
   * `docker/homeassistant/secrets.yaml.example` → `docker/homeassistant/secrets.yaml`
   * `docker/esphome/secrets.yaml.example` → `docker/esphome/secrets.yaml`
2. Fill in real values.
3. From the repo root:

```bash
docker compose -f docker/docker-compose.yml up -d
```

On the **homelab server**, production bind mounts may still live under `~/docker/`; see `AGENT_HANDOFF.md` and `INFRASTRUCTURE.MD` before assuming paths.

## Notes

Review before publishing; this repo is intended to stay public-safe.
