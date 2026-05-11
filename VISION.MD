# VISION.md

# 3D Printing Farm Homelab Vision

## Overview

This document defines the long-term vision, architecture, operational philosophy, and infrastructure goals for the 3D printing farm homelab.

The system is intended to become:

* A scalable multi-printer orchestration environment
* A centralized monitoring and automation platform
* A self-hosted infrastructure lab
* A development environment for custom printer farm tooling
* A reliable remote-management environment for printers and supporting services

The stack prioritizes:

* Reliability
* Recoverability
* Observability
* Remote management
* Containerization
* Reproducibility
* Ease of maintenance
* Future scalability

## Documentation Model

This repository uses two complementary infrastructure documents:

* `VISION.MD` for long-term architecture, goals, and future-state planning
* `INFRASTRUCTURE.MD` for the currently deployed stack, image versions, ports, storage, and operational procedures

When the live environment changes, update `INFRASTRUCTURE.MD`.

When the target architecture or long-term direction changes, update `VISION.MD`.

---

# Current Hardware

## Primary Server

Laptop-based Ubuntu Server host.

### Current Specs

* Ubuntu Server
* Ethernet-connected
* Docker installed
* 2-drive capable laptop platform
* SSD-based storage
* Headless/server-oriented operation

### Storage Strategy

Current plan:

* Drive 1:

  * Ubuntu Server
  * Docker
  * Containers
  * Configuration
  * Core services

* Drive 2 (future/planned):

  * Nextcloud data
  * Timelapses
  * Printer recordings
  * Backups
  * Persistent orchestrator data
  * Shared storage

---

# Core Infrastructure Philosophy

## Everything Containerized

The infrastructure should favor Docker containers over direct host installations whenever possible.

Benefits:

* Easier backups
* Easier migrations
* Easier upgrades
* Better isolation
* Reproducible deployments
* Cleaner maintenance

Exceptions should be minimal and deliberate.

---

# Base Operating System

## Ubuntu Server

Chosen over Alpine Linux and NAS-oriented systems because:

* Better compatibility
* Better Docker ecosystem support
* Better AI/tutorial ecosystem support
* Easier troubleshooting
* More predictable infrastructure behavior
* Better support for monitoring stacks and orchestration tooling

---

# Current Installed Services

## Docker

Primary container runtime.

### Goals

* Docker-first architecture
* Compose-based deployments
* Reproducible environments
* Standardized container structure

---

## Portainer

Container management dashboard.

### Purpose

* Container visibility
* Stack management
* Logs
* Restarts
* Quick operational insight

### Planned Usage

* Infrastructure monitoring
* Stack deployment management
* Quick diagnostics
* Operational dashboard

---

## Home Assistant

Central automation platform.

### Purpose

* Power control
* Automation orchestration
* Device state monitoring
* Farm automation
* Alerting integrations

### Future Automation Examples

* Auto power-cycle failed printers
* Power management schedules
* Emergency shutdown automation
* Environmental monitoring
* Print completion workflows

---

## ESPHome

Firmware and device management platform.

### Purpose

* ESP device management
* Relay control
* OTA firmware updates
* Power monitoring
* Sensor integration

### Planned Hardware

* ESP32 relay controllers
* Power monitoring modules
* Environmental sensors
* Printer power control

---

# Remote Access & External Connectivity

## Tailscale VPN

The infrastructure will use Tailscale as the primary remote-access solution.

### Goals

* Secure remote SSH access
* Remote infrastructure management
* Access to internal dashboards and services
* Avoid direct port exposure whenever possible
* Simplify remote administration

### Planned Usage

* Server joins existing Tailscale network
* SSH access over Tailscale
* Remote access to Portainer
* Remote access to Home Assistant
* Remote access to monitoring dashboards
* Potential orchestrator access

### Philosophy

Tailscale should be considered the primary management-access layer.

Public exposure should be minimized whenever possible.

---

## SSL & Domain Strategy

An AWS-managed domain will be used for externally exposed services.

### Planned Approach

* Create subdomains for exposed services
* Use reverse proxy for SSL termination
* Restrict exposure to only necessary services
* Maintain most infrastructure behind Tailscale

### Potential Exposed Services (TBD)

Possible candidates:

* Nextcloud
* Monitoring dashboards
* Orchestrator UI
* Selected printer interfaces

### Potential Reverse Proxy Options

* Traefik
* NGINX Proxy Manager
* NGINX

### SSL Goals

* HTTPS everywhere
* Centralized routing
* Cleaner service URLs
* Easier external access management

---

# Printer Naming Strategy

## Numeric IDs

Printers will maintain stable numeric identifiers.

### Format

* printer01
* printer02
* printer03
* printer04
* printer05
* printer06

These identifiers should remain stable internally for:

* Infrastructure
* Compose stacks
* Automation
* Metrics
* APIs
* Internal orchestration

---

## Friendly Names

Printers will also receive human-friendly names inspired by the seven dwarf lords from The Lord of the Rings.

The farm will use six of the seven names.

### Goals

* Easier operational identification
* Better UI/UX
* More memorable printer references
* Human-friendly dashboards

### Usage Philosophy

Infrastructure should support both:

* Stable numeric identifiers
* Friendly display names

Examples:

* printer01 (friendly: TBD)
* printer02 (friendly: TBD)

Friendly names should be treated as metadata/display labels, not primary infrastructure identifiers.

---

# Planned Infrastructure Services

## Printer Infrastructure

### Target

6 independent printer stacks.

Each printer stack will contain:

* Klipper
* Moonraker
* Mainsail
* ustreamer

### Goals

* Isolation between printers
* Standardized deployment templates
* Easy replacement/recovery
* Scalable management
* Central orchestration

---

## Custom Printer Orchestrator

Custom-built orchestration platform.

### Purpose

* Multi-printer coordination
* Queue management
* Fleet health monitoring
* Farm automation
* Centralized operations
* Print workflow automation
* Metrics collection
* Operational analytics

### Long-Term Vision

Potentially evolve into:

* Full printer farm management platform
* Distributed print scheduling system
* Commercial/internal orchestration layer

---

## Nextcloud

Self-hosted storage and collaboration platform.

### Planned Uses

* Shared files
* STL repository
* Timelapse storage
* Documentation
* Backups
* Internal file synchronization
* Project asset management

---

## Monitoring Stack

### Planned Components

* Uptime Kuma
* Prometheus
* Grafana
* Alertmanager

### Goals

* Printer uptime monitoring
* Container health monitoring
* Resource usage tracking
* Alerting
* Historical metrics
* Capacity planning
* Operational dashboards

---

## MQTT Broker

Likely Mosquitto-based.

### Purpose

* ESPHome communication
* Home Assistant messaging
* Event distribution
* Future orchestrator integration

---

## Reverse Proxy

Potential options:

* Traefik
* NGINX Proxy Manager
* NGINX

### Goals

* Unified access layer
* SSL termination
* Cleaner routing
* Future external access
* Easier service discovery

---

## Plex (Optional)

Optional media infrastructure.

Not considered core to the printer farm.

Potential secondary-use service.

---

# Networking Strategy

## Ethernet-First Server

The main server should operate primarily over Ethernet.

Reasons:

* Reliability
* Stable latency
* Better throughput
* More predictable discovery behavior
* Better support for infrastructure workloads

---

## WiFi Devices

ESPHome devices will operate over WiFi.

### Requirements

* Same subnet as server
* DHCP reservations/static IPs
* Reliable 2.4GHz coverage

---

## Static DHCP Reservations

All critical infrastructure devices should receive reserved IP addresses.

Examples:

* Server
* ESP controllers
* Cameras
* Printers
* Future NAS/storage devices

---

# Naming Conventions

## Printers

* printer01
* printer02
* printer03
* etc.

## ESP Devices

* esp-printer01-power
* esp-printer02-power
* etc.

## Containers

* klipper-printer01
* moonraker-printer01
* mainsail-printer01
* ustreamer-printer01

---

# Operational Goals

## Reliability

The farm should:

* Recover cleanly from reboots
* Minimize manual intervention
* Support unattended operation
* Be remotely manageable

---

## Observability

The system should provide:

* Health dashboards
* Logs
* Metrics
* Alerts
* Resource visibility
* Service visibility

---

## Scalability

The architecture should support:

* Additional printers
* Additional ESP devices
* Additional cameras
* Additional services
* Potential migration to larger hardware later

---

## Recoverability

The system should support:

* Easy redeployment
* Config backups
* Persistent data separation
* Infrastructure-as-code direction
* Minimal downtime recovery

---

# Security Considerations

## Initial Philosophy

Prioritize:

* Simplicity
* Operational stability
* Internal-network security

Before introducing:

* Public exposure
* Complex auth layers
* Internet-facing services

---

## Future Security Enhancements

Potential future additions:

* VPN access
* Reverse proxy auth
* SSL certificates
* VLAN separation
* Backup automation
* External monitoring

---

# Long-Term Vision

This homelab is intended to evolve from:

* A simple multi-printer setup

Into:

* A fully managed printer farm platform
* A development and experimentation environment
* A self-hosted infrastructure ecosystem
* A scalable orchestration platform

The infrastructure should remain:

* Modular
* Well-documented
* Easy to maintain
* Easy to expand
* Docker-first
* Automation-oriented

---

# Immediate Next Steps

## Infrastructure

* Finalize Home Assistant setup
* Finalize ESPHome setup
* Configure relay hardware
* Validate OTA updates
* Configure static DHCP leases

## Monitoring

* Deploy Uptime Kuma
* Begin health monitoring
* Configure alerts

## Printer Layer

* Create reusable printer stack templates
* Define printer compose architecture
* Standardize volume structure
* Standardize networking

## Orchestrator

* Begin architecture definition
* Define service responsibilities
* Define printer communication layer
* Define monitoring strategy

---

# Guiding Principle

The goal is not just to host services.

The goal is to build:

* A reliable infrastructure platform
* A remotely manageable printer farm
* A scalable automation environment
* A foundation for custom tooling and orchestration
