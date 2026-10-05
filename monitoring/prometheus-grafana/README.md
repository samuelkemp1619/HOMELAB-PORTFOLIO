# Homelab Monitoring — Prometheus, Grafana & Dashy

## Overview

This project documents the centralized monitoring environment used to observe the health and availability of my homelab infrastructure.

The monitoring stack runs within a dedicated Linux container on the Proxmox cluster and provides metrics collection, visualization, dashboards, and a foundation for automated health reporting and centralized log analysis.

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

Monitoring targets include or are being expanded to include:

- Proxmox hosts
- Virtual machines
- LXC containers
- Network services
- DNS
- Critical applications
- Service availability

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

## Exception-Based Monitoring

A major goal of the monitoring environment is to reduce unnecessary information and emphasize conditions that require attention.

Normal CPU, memory, and storage utilization generally does not need to appear in routine reports.

Instead, abnormal conditions should be identified based on factors such as:

- Severity
- Duration
- Deviation from normal behavior
- Service impact

Examples include:

- Sustained high CPU usage
- Abnormal memory pressure
- Storage approaching capacity
- Unexpected service outages
- Packet loss
- Elevated network latency
- DNS failures
- Backup failures
- Cluster quorum problems

## Centralized Logging

Centralized log collection is planned using Grafana Loki or a similar logging platform.

Relevant logs will be collected from infrastructure such as:

- Proxmox
- Linux systems
- OPNsense
- Pi-hole
- Critical services

Rather than reporting every warning or error, the system will prioritize noteworthy events and suppress repetitive noise.

Events may be categorized as:

- Critical
- Warning
- Security
- Network
- Service
- Backup
- Hardware

## Automated Health Reporting

A planned reporting service will combine monitoring metrics and log analysis to generate scheduled HTML email health reports.

Reports will focus on exceptions rather than routine metrics.

Planned report sections include:

- Proxmox cluster and quorum health
- Critical service status
- Network and gateway health
- DNS health
- Tailscale connectivity
- Backup status
- Resource anomalies
- Unusual log events
- Security events
- Severity summary
- Recommended attention

Critical failures may generate immediate notifications rather than waiting for the scheduled report.

## Backup & Recovery

The monitoring container is backed up using Proxmox `vzdump`.

Future improvements include adding the monitoring workload to scheduled off-node backups and performing documented restore testing.

## Skills Demonstrated

- Infrastructure monitoring
- Prometheus
- Grafana
- Linux administration
- LXC administration
- Time-series metrics
- Dashboard development
- Service monitoring
- Log analysis
- Alert design
- Observability
- Troubleshooting

## Future Improvements

- Deploy centralized logging with Grafana Loki
- Implement exception-based HTML email reports
- Add immediate critical alerts
- Expand Prometheus targets
- Monitor backup health
- Add network latency and packet-loss monitoring
- Add DNS anomaly detection
- Establish baseline behavior for anomaly detection
- Document dashboards and alert thresholds
- Add sanitized screenshots to this repository
