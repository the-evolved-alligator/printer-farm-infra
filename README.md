# Printer Farm Infrastructure

Public-safe infrastructure repository for the 3D printing farm homelab.

This repo contains:
- `VISION.MD` for long-term architecture and roadmap
- `INFRASTRUCTURE.MD` for the current deployed stack and operations
- `docker/docker-compose.yml` for the base infrastructure stack
- sanitized `Home Assistant` and `ESPHome` configuration files
- example secret files instead of live credentials

## Included services

- `Home Assistant`
- `ESPHome`
- `Uptime Kuma`
- `Portainer`

## Excluded from Git

The live repo intentionally excludes:
- runtime state and databases
- logs and lock files
- actual secret files
- Home Assistant `.storage` and cache directories
- ESPHome build artifacts

## Setup

1. Copy the example secret files:
   - `docker/homeassistant/secrets.yaml.example` to `docker/homeassistant/secrets.yaml`
   - `docker/esphome/secrets.yaml.example` to `docker/esphome/secrets.yaml`
2. Replace placeholder secret values.
3. Review `docker/docker-compose.yml` for your host.
4. Start the stack:

```bash
docker compose -f docker/docker-compose.yml up -d
```

## Notes

This repo is prepared to be public-safe, but review every file before publishing.
