# Directory layout

This document explains where everything lives on the homelab server and how it relates to the Git repository.

## Canonical Git repository

| Path | Role |
|------|------|
| `/home/daniel-quicazan/printer-farm-infra` | Single source of truth for **documentation**, **sanitized** ESPHome/Home Assistant templates, and Compose **as committed to GitHub**. |

Remote: `git@github.com:the-evolved-alligator/printer-farm-infra.git`

## Live Docker runtime (bind mounts)

| Path | Role |
|------|------|
| `/home/daniel-quicazan/docker/docker-compose.yml` | Compose file **actually used** by `docker compose` on this host. |
| `/home/daniel-quicazan/docker/homeassistant` | Live Home Assistant `/config` volume (includes `.storage`, DB, logs; not in Git). |
| `/home/daniel-quicazan/docker/esphome` | Live ESPHome `/config` (device YAML, `secrets.yaml`; real secrets **not** in Git). |

## Home directory pointers

At `/home/daniel-quicazan`:

* `VISION.MD` and `INFRASTRUCTURE.MD` are **symlinks** into `printer-farm-infra/` so there is no duplicate copy to drift.

## What not to move blindly

* Do **not** relocate `/home/daniel-quicazan/docker/` without updating Compose volume paths and restarting the stack.
* Do **not** commit `secrets.yaml`, `.storage/`, databases, or logs from the live `docker/` tree.

## Optional future layout

If you later want one tree only, you could move live `docker/` under the repo and use long-lived symlinks, or use a deploy script that copies from repo to `~/docker`. That was **not** done yet; current split is intentional and documented in `AGENT_HANDOFF.md`.
