# Homelab Monitoring — Prometheus, Grafana & Dashy

## Dashboard Overview

![Illustrative Proxmox monitoring dashboard composite](Proxmox%20Monitoring%20Dashboard%20Overview.png)

*Illustrative composite reconstructed from multiple Grafana dashboard screenshots. It shows the types of metrics tracked—including node health, CPU, memory, disk, VM/LXC status, network traffic, storage, and backup coverage—but is not a pixel-exact capture or a verified source of metric values. Some labels and values may differ from the original monitoring data.*

**Publication note:** The uploaded image contains internal node aliases/names and workload identifiers. Treat these as potentially identifying infrastructure details; replace them with generic labels before using the image as a fully sanitized portfolio visual.

## Overview

This project documents the centralized monitoring environment used to observe the health and availability of my homelab infrastructure.

The monitoring stack runs within a dedicated Linux container on the Proxmox cluster and provides metrics collection, visualization, dashboards, and service-health dashboards.

## Technologies

- Prometheus
- Grafana
- Dashy
- Proxmox VE
- Linux Containers (LXC)
- Linux
- Raspberry Pi kiosk display

## Architecture

The monitoring services run in a dedicated Proxmox LXC container.

This separates monitoring from the physical Proxmox hosts while providing centralized visibility into the environment.

General architecture:

Infrastructure & Services
        ↓
    Prometheus
        ↓
     Grafana
        ↓
 Dashboards / Reporting

Dashy provides an additional centralized dashboard for accessing homelab services.

## Prometheus

Prometheus provides time-series metric collection for the environment.

The monitoring system is designed to observe infrastructure health while retaining historical information that can be used for troubleshooting and trend analysis.

Monitoring covers Proxmox hosts, virtual machines, containers, and service metrics through configured exporters.

## Grafana

Grafana provides visualization of collected monitoring data.

Dashboards are used to provide visibility into:

- Infrastructure health
- Host availability
- Service availability
- Network behavior
- Resource utilization
- Historical trends

Resource utilization is primarily useful for identifying abnormal conditions rather than requiring constant manual review.

## Dashy

Dashy provides a centralized interface for accessing homelab services and monitoring resources.

It runs alongside the monitoring environment while remaining logically separate from Prometheus metric collection and Grafana visualization.

## Raspberry Pi Monitoring Display

A Raspberry Pi is used as a dedicated physical monitoring display.

The Pi operates as a kiosk-style dashboard display, allowing Grafana dashboards to remain visible without requiring a workstation to remain dedicated to monitoring.

This creates a physical Network Operations Center-style status display for the homelab.

## Backup & Recovery

The monitoring container is backed up using Proxmox `vzdump`.

Backup and recovery readiness are tracked separately from this completed monitoring deployment.

## Skills Demonstrated

- Infrastructure monitoring
- Prometheus
- Grafana
- Linux administration
- LXC administration
- Time-series metrics
- Dashboard development
- Service monitoring
- Observability
- Troubleshooting

