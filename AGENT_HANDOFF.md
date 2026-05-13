# Agent handoff — printer farm homelab

Read this file first when continuing work in this environment. It captures decisions, live paths, and what is done vs pending.

## Machine

* Hostname: `quicazan-rubio-homelab`
* User: `daniel-quicazan`
* OS: Ubuntu Server, Docker + Docker Compose
* Remote access: Tailscale active on host (not in Compose)
* Timezone in stack: `America/Bogota` (see `docker-compose.yml`)

## Repository vs runtime (critical)

| Concern | Location |
|---------|----------|
| **Git / GitHub** | `/home/daniel-quicazan/printer-farm-infra` → `git@github.com:the-evolved-alligator/printer-farm-infra.git` |
| **Running Docker stack** | `/home/daniel-quicazan/docker/` — this is what `docker compose` uses today |

Docs in the repo describe both; see `INFRASTRUCTURE.MD` section **Live host paths**.

Root-level `VISION.MD` and `INFRASTRUCTURE.MD` under `/home/daniel-quicazan` are **symlinks** into `printer-farm-infra/` to avoid duplicate docs.

## Base stack (deployed)

Compose project name: `infrastructure`. Services:

| Service | Image (pinned) | Notes |
|---------|----------------|--------|
| Home Assistant | `ghcr.io/home-assistant/home-assistant:2026.5.1` | `network_mode: host`, UI `8123` |
| ESPHome | `ghcr.io/esphome/esphome:2026.4.5` | `network_mode: host`, UI `6052` |
| Uptime Kuma | `louislam/uptime-kuma:1.23.17` | Port `3001`, volume `uptime-kuma` |
| Portainer | `portainer/portainer-ce:2.39.2` | Ports `9443` (HTTPS), `8000` |

Compose file (live): `/home/daniel-quicazan/docker/docker-compose.yml`

External named volumes: `uptime-kuma`, `portainer_data`.

## Printers / Klipper (MCU USB)

* Boards: **BTT SKR 3** (non-EZ), MCU **STM32H723** (chip marking).
* Correct Klipper `menuconfig` baseline: STM32 → **STM32H723**, **128KiB bootloader**, **25 MHz crystal**, **USB (PA11/PA12)**. Wrong MCU (e.g. F103) = no USB serial device.
* After correct firmware: host shows **Klipper** USB devices (`idVendor=1d50`, `idProduct=614e`) and `/dev/serial/by-id/usb-Klipper_stm32h723xx_<serial>-if00`.

**Operational check:**

```bash
ls -l /dev/serial/by-id
ls /dev/ttyACM*
```

There are **six** printers; each board has a **unique** `by-id` symlink. Map `printer01`–`printer06` to those paths manually (label cables or unplug one at a time) — not stored in this repo yet.

## ESPHome

* Live configs: `/home/daniel-quicazan/docker/esphome/` (includes `esp-printer01-power.yaml` … `06`, and `secrets.yaml`).
* Public templates + `secrets.yaml.example`: under `printer-farm-infra/docker/esphome/`.

## Git / SSH

* `gh` CLI was not installed; remote added with `git remote`.
* Initial push required fixing `~/.ssh/id_rsa` permissions (`chmod 600`).
* Default branch: `main`.

## Documentation map

| File | Purpose |
|------|---------|
| `VISION.MD` | Long-term architecture and roadmap |
| `INFRASTRUCTURE.MD` | Current stack, ports, backup/update procedures, live vs repo paths |
| `docs/DIRECTORY_LAYOUT.md` | Where folders live on disk |
| `AGENT_HANDOFF.md` | This file — session continuity for agents |
| `README.md` | Repo overview and clone setup |

## Completed in prior work (summary)

* Base services moved under Compose with project name `infrastructure`, pinned images, dedicated bridge network for Portainer/Kuma.
* Public-safe repo created; secrets redacted in Git; `.gitignore` excludes live secrets and HA/ESPHome runtime noise.
* GitHub remote configured; initial commit pushed after SSH fix.
* Klipper MCU visibility issue traced to wrong firmware target; **STM32H723** build confirmed working (USB + `cdc_acm`).
* Six MCUs enumerated successfully when all printers connected.

## Suggested next steps (not done)

1. **Printer ↔ serial map**: document which `by-id` path belongs to `printer01` … `printer06` (table in `INFRASTRUCTURE.MD` or a new `docs/PRINTER_USB_MAP.md` — keep repo free of sensitive layout if needed).
2. **Per-printer Docker stacks**: Klipper, Moonraker, Mainsail, ustreamer per `VISION.MD` (orchestrator is separate future work).
3. **Sync policy**: when changing live `/home/daniel-quicazan/docker/docker-compose.yml`, mirror into `printer-farm-infra/docker/docker-compose.yml` and commit, or script the copy.
4. **Uptime Kuma**: add monitors for each service and each printer once Moonraker exists.
5. **Optional**: `backup-infrastructure.sh` next to documented tar flow in `INFRASTRUCTURE.MD`.

## Safety

* Never paste or commit live WiFi, API keys, or OTA passwords from `docker/esphome/secrets.yaml` or device YAMLs.
* Live Home Assistant `secrets.yaml` is also git-ignored in the repo templates workflow.
