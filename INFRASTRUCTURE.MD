# INFRASTRUCTURE.md

# 3D Printing Farm Homelab Infrastructure

This document describes the infrastructure that is actually deployed today, how it is organized, and how to operate it.

`VISION.MD` is the long-term plan.

`INFRASTRUCTURE.MD` is the current-state reference.

---

## Purpose

This file is the operational source of truth for the homelab base infrastructure.

Use it to answer:

* What is currently deployed
* Where each service stores data
* How services are exposed
* How the stack is managed
* What is still missing relative to the long-term vision

---

## Current Host

Primary host:

* Ubuntu Server
* Docker installed
* Docker Compose in use
* Ethernet-connected
* Tailscale active on the host

Repository paths (clone of this repo, templates and public-safe configs):

* Compose stack: `docker/docker-compose.yml`
* Home Assistant config: `docker/homeassistant`
* ESPHome config: `docker/esphome`

### Live host paths (this server)

On `quicazan-rubio-homelab`, Docker bind mounts and the **running** Compose stack use the home directory, not the repo checkout:

* Running Compose file: `/home/daniel-quicazan/docker/docker-compose.yml`
* Live Home Assistant data: `/home/daniel-quicazan/docker/homeassistant`
* Live ESPHome data: `/home/daniel-quicazan/docker/esphome`

Operational commands on this machine must use the live path, for example:

```bash
docker compose -f /home/daniel-quicazan/docker/docker-compose.yml ps
```

The Git repository at `/home/daniel-quicazan/printer-farm-infra` is the **version-controlled** copy: docs, sanitized configs, and `docker-compose.yml` kept in sync with production when you deliberately update both.

---

## Current Stack Identity

Compose project name:

* `infrastructure`

Docker bridge network:

* `infrastructure`

Notes:

* `Home Assistant` and `ESPHome` use `host` networking
* `Portainer` and `Uptime Kuma` use the dedicated bridge network above

---

## Deployed Services

### Home Assistant

Role:

* Central automation platform

Status:

* Deployed
* Persistent
* Initialized

Image:

* `ghcr.io/home-assistant/home-assistant:2026.5.1`

Container:

* `homeassistant`

Networking:

* Host network
* Expected local UI: `http://<host-ip>:8123`

Persistent data:

* `./docker/homeassistant:/config`

Current maturity:

* Base installation is working
* Onboarding completed
* YAML config is still minimal
* `automations.yaml` is currently empty

---

### ESPHome

Role:

* ESP device dashboard and firmware management

Status:

* Deployed
* Persistent
* Healthy

Image:

* `ghcr.io/esphome/esphome:2026.4.5`

Container:

* `esphome`

Networking:

* Host network
* Expected local UI: `http://<host-ip>:6052`

Persistent data:

* `./docker/esphome:/config`

Current maturity:

* Dashboard is running
* No device YAML definitions are present yet

---

### Uptime Kuma

Role:

* Infrastructure health and uptime monitoring

Status:

* Deployed
* Persistent
* Healthy

Image:

* `louislam/uptime-kuma:1.23.17`

Container:

* `uptime-kuma`

Networking:

* Published port `3001`
* Local UI: `http://<host-ip>:3001`

Persistent data:

* Docker volume: `uptime-kuma`

Current maturity:

* Application is initialized
* Database exists
* Monitor count is currently `0`

---

### Portainer

Role:

* Docker and stack management UI

Status:

* Deployed
* Persistent
* Running

Image:

* `portainer/portainer-ce:2.39.2`

Container:

* `portainer`

Networking:

* Published ports `8000` and `9443`
* Preferred local UI: `https://<host-ip>:9443`

Persistent data:

* Docker volume: `portainer_data`
* Docker socket bind mount: `/var/run/docker.sock:/var/run/docker.sock`

Current maturity:

* Operational
* Persistent state exists

---

## Host-Level Services

### Tailscale

Role:

* Secure remote management access

Status:

* Active on the host

Notes:

* This is currently host-managed, not containerized
* It is part of the real deployed infrastructure even though it is not in Compose

---

## Not Yet Deployed

The following items are part of the broader vision but are not currently visible on this host:

* Printer stacks (`Klipper`, `Moonraker`, `Mainsail`, `ustreamer`)
* Custom printer orchestrator
* `Mosquitto` / MQTT broker
* `Prometheus`
* `Grafana`
* `Alertmanager`
* `Nextcloud`
* Reverse proxy (`Traefik`, `NGINX Proxy Manager`, or `NGINX`)

---

## Ports

Current exposed service ports:

* `8123` - Home Assistant
* `6052` - ESPHome
* `3001` - Uptime Kuma
* `9443` - Portainer HTTPS
* `8000` - Portainer tunnel

---

## Storage

### Bind-mounted data

* `docker/homeassistant`
* `docker/esphome`

### Named Docker volumes

* `uptime-kuma`
* `portainer_data`

### Important note

Files inside bind-mounted config directories may be owned by `root` because the containers write them directly.

---

## Operations

Primary stack file:

* `docker/docker-compose.yml`

Common commands:

```bash
docker compose -f docker/docker-compose.yml up -d
docker compose -f docker/docker-compose.yml ps
docker compose -f docker/docker-compose.yml logs -f
docker compose -f docker/docker-compose.yml pull
docker compose -f docker/docker-compose.yml up -d
```

Service-specific logs:

```bash
docker compose -f docker/docker-compose.yml logs -f homeassistant
docker compose -f docker/docker-compose.yml logs -f esphome
docker compose -f docker/docker-compose.yml logs -f uptime-kuma
docker compose -f docker/docker-compose.yml logs -f portainer
```

Stop the stack:

```bash
docker compose -f docker/docker-compose.yml down
```

---

## Backups And Updates

### Backup priorities

Back up these items first:

* `docker/homeassistant`
* `docker/esphome`
* Docker volume `uptime-kuma`
* Docker volume `portainer_data`

### Simple backup workflow

This is a conservative manual backup flow for the current stack:

```bash
BACKUP_DIR="${PWD}/backups/infrastructure-$(date +%F-%H%M)"
mkdir -p "$BACKUP_DIR"

docker compose -f docker/docker-compose.yml stop

tar -czf "$BACKUP_DIR/homeassistant.tar.gz" -C . docker/homeassistant
tar -czf "$BACKUP_DIR/esphome.tar.gz" -C . docker/esphome

docker run --rm -v uptime-kuma:/source -v "$BACKUP_DIR:/backup" alpine \
  tar -czf /backup/uptime-kuma.tar.gz -C /source .

docker run --rm -v portainer_data:/source -v "$BACKUP_DIR:/backup" alpine \
  tar -czf /backup/portainer_data.tar.gz -C /source .

docker compose -f docker/docker-compose.yml up -d
```

### Restore notes

Restore requires:

* The bind-mounted directories placed back under `docker/`
* The named Docker volumes recreated and repopulated if they are lost
* The stack brought back with Compose after the data is restored

### Update workflow

For routine container updates:

```bash
docker compose -f docker/docker-compose.yml pull
docker compose -f docker/docker-compose.yml up -d
docker compose -f docker/docker-compose.yml ps
```

Recommended update order:

* Confirm `INFRASTRUCTURE.MD` reflects the current stack before changing anything
* Take a backup before major updates
* Update one logical stack at a time
* Verify each UI after the update
* Record the new image versions in this file if they change

---

## Change Management

When the infrastructure changes, update this file with:

* New services
* Removed services
* Image version changes
* Port changes
* Volume changes
* New operational commands
* New host-level dependencies

---

## Relationship To Vision

`VISION.MD` should answer:

* Where the platform is going
* Why the architecture exists
* What still needs to be built

`INFRASTRUCTURE.MD` should answer:

* What is deployed right now
* How it is wired today
* How to operate and troubleshoot it today
