# Graphy — Raspberry Pi Grafana Monitoring Kiosk

**Status:** Completed — October 3, 2026  
**Category:** Monitoring / Linux / Embedded Infrastructure

## Overview

Deployed a Raspberry Pi 4 Model B as a dedicated always-on dashboard display for the homelab's existing Grafana monitoring environment. The kiosk uses Chromium under labwc and displays a Grafana playlist rotating dashboards every 30 seconds.

## Implementation

- Raspberry Pi 4 Model B, named **Graphy**
- Wi-Fi connection on the trusted VLAN 20
- Linux graphical session using **labwc**
- Chromium configured for kiosk-style dashboard viewing
- Grafana playlist configured to rotate at **30-second intervals**
- Existing Prometheus/Grafana monitoring stack provides dashboard data

## Completed Work

- [x] Set up the Raspberry Pi as a dedicated dashboard appliance
- [x] Configured graphical kiosk display using labwc and Chromium
- [x] Connected the device to the appropriate trusted network
- [x] Configured and verified Grafana playlist rotation
- [x] Completed dashboard deployment

## Architecture

`Prometheus / infrastructure metrics → Grafana dashboards → Grafana playlist → Chromium kiosk on Graphy`

Graphy is a **display endpoint**, not a replacement for the monitoring servers.

## Skills Demonstrated

Linux configuration, Raspberry Pi administration, monitoring visualization, browser kiosk deployment, Wi-Fi/VLAN integration, and systems troubleshooting.

## Notes

Screenshots and sanitized configuration examples can be added later. This project is complete; separate centralized logging and exception-based reporting work remains paused.
