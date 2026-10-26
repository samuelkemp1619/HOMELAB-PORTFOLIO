# Proxmox VE Homelab Cluster

## Overview

This project documents the design, deployment, and administration of my four-node Proxmox VE cluster.

The cluster provides the virtualization platform for my homelab and hosts networking, DNS, monitoring, game-server, and other infrastructure services.

## Cluster Hardware

| Node | Hardware | Primary Role |
|------|----------|--------------|
| HomeProx | Dell OptiPlex 7020 Micro | Primary Infrastructure |
| HomeProx2 | HP Elite Mini 600 G9 | Application Host |
| HomeProx3 | HP Elite Mini 600 G9 | Container Host |
| HomeProx4 | Dell PowerEdge R640 | Windows Server and AI-Operations Host |

## Technologies

- Proxmox VE
- Debian Linux
- KVM/QEMU
- Linux Containers (LXC)
- Corosync
- LVM-Thin
- NFS
- VLAN-aware Linux bridges
- Automated backups

## Current Workloads

### HomeProx
- OPNsense virtual firewall/router
- Pi-hole DNS filtering

### HomeProx2
- AMP game-server management (CT 102)
- Dashy, Prometheus and Grafana monitoring stack (CT 103)

### HomeProx3
- Vaultwarden (CT 104)

### HomeProx4
- Windows Server 2025 (VM 400)
- AI-Operations monitoring and automation (VM 410)

## Cluster Design

The four Proxmox nodes operate as a single cluster using Corosync for cluster communication and quorum.

The cluster has four voting nodes. Quorum requires three votes; loss of one node can be tolerated while retaining quorum.

## Networking

Proxmox uses VLAN-aware networking to support segmented networks within the homelab.

Current network segmentation includes dedicated management and trusted networks, with additional segmentation planned for services and security labs.

Routing and firewall policy are handled by OPNsense.

## October 2026 Cluster Maintenance

- Confirmed four-node quorum and documented VM/LXC placement across hosts.
- Migrated CT 103 (Dashy, Prometheus, Grafana and related exporters) to HomeProx2 and verified network and dashboard access afterward.
- Configured two scheduled `vzdump` jobs on off-node `infra-backups` storage: a 03:00 snapshot job for VM 100 and CT 101, and a 03:30 stop-mode job for CTs 102, 103 and 104.
- Both jobs use ZSTD compression and retention of 3 last, 7 daily, 4 weekly and 3 monthly backups. Successful recent backup tasks were observed; this is not a claim of a completed restore test.
- Windows Server 2025 VM 400 is installed and activated; further directory-services configuration is paused.

## Backup Strategy

Proxmox `vzdump` is used to create compressed VM and LXC backups.

Backup monitoring and expanded coverage are ongoing projects, including:

- Scheduled backups
- Backup retention policies
- Off-node storage
- Restore testing
- Backup failure monitoring
- Automated health reporting

## Monitoring

The environment is monitored using Prometheus and Grafana.

Monitoring includes infrastructure availability and performance, with AI-assisted exception detection and alert routing implemented in n8n. Pushover delivery testing and scheduled activation remain pending. Centralized logging remains future work.

## Administration

Administrative tasks performed in this environment include:

- Cluster deployment and maintenance
- Rolling Proxmox upgrades
- Kernel updates
- VM and LXC management
- Storage management
- Backup and recovery
- VLAN configuration
- Linux troubleshooting
- Cluster quorum verification
- Service availability validation

## Skills Demonstrated

- Virtualization administration
- Linux systems administration
- High-availability concepts
- Cluster management
- Network segmentation
- Storage administration
- Backup and recovery
- Infrastructure monitoring
- Change management
- Troubleshooting

## Future Improvements

- Expand automated backup coverage
- Perform documented restore testing
- Improve centralized logging
- Implement exception-based health reports
- Expand security monitoring
- Document disaster-recovery procedures
