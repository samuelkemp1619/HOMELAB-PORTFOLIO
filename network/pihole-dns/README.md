# Pi-hole DNS Filtering

## Overview

This project documents the deployment and administration of Pi-hole as the centralized DNS filtering service for my homelab.

Pi-hole runs as a Linux container within the Proxmox cluster and provides DNS-based filtering, visibility into DNS activity, and centralized DNS management for network clients.

## Technologies

- Pi-hole
- Proxmox VE
- Linux Containers (LXC)
- OPNsense
- DNS
- VLANs
- Cloudflare DNS
- Prometheus / Grafana

## Architecture

Pi-hole is deployed as a dedicated LXC container rather than being installed directly on a Proxmox host.

This keeps the DNS service isolated from the hypervisor and allows it to be independently:

- Backed up
- Updated
- Monitored
- Restarted
- Troubleshot
- Restored

OPNsense provides routing and firewall services while Pi-hole provides DNS filtering.

## DNS Flow

The general DNS path is:

Client Device → Network → Pi-hole → Upstream DNS

Pi-hole evaluates DNS requests against configured filtering rules before forwarding permitted queries to upstream DNS resolvers.

## DNS Filtering

Pi-hole provides centralized filtering for devices throughout the network.

This allows filtering policies to be applied without installing filtering software individually on every supported client.

The service also provides visibility into:

- DNS query volume
- Blocked queries
- Frequently requested domains
- Client DNS activity
- Upstream DNS behavior

## Network Integration

Pi-hole operates alongside the segmented network architecture managed by OPNsense.

DNS availability and firewall policy must be considered when introducing or modifying VLANs so that authorized clients can reach DNS while unnecessary inter-VLAN access remains restricted.

## Monitoring

Pi-hole is incorporated into the homelab monitoring environment.

Monitoring and reporting are being expanded to include:

- DNS service availability
- Query activity
- Blocking activity
- Upstream resolver health
- DNS response failures
- Abnormal DNS behavior
- Service interruptions

## Backup & Recovery

The Pi-hole container is included in scheduled Proxmox backups.

Backups use compressed Proxmox `vzdump` archives with retention policies to maintain multiple recovery points.

Future work includes documented restore testing and additional DNS redundancy.

## Troubleshooting

Administration of the Pi-hole deployment has included troubleshooting:

- DNS resolution failures
- Upstream resolver configuration
- Network connectivity
- Time synchronization
- Pi-hole upgrades
- API changes
- Dashboard integration
- Cross-origin API requests
- Client performance issues

## Security Considerations

DNS is a critical infrastructure service, so the deployment is designed to minimize unnecessary exposure.

Security considerations include:

- Restricting administrative access
- Network segmentation
- Maintaining current software
- Monitoring DNS availability
- Maintaining backups
- Avoiding unnecessary Internet exposure
- Monitoring abnormal DNS behavior

## Skills Demonstrated

- DNS administration
- Linux administration
- LXC container management
- Network troubleshooting
- DNS filtering
- OPNsense integration
- Service monitoring
- Backup and recovery
- Infrastructure troubleshooting

## Future Improvements

- Deploy secondary DNS for redundancy
- Improve DNS health monitoring
- Add centralized Pi-hole log collection
- Detect unusual DNS activity
- Integrate DNS anomalies into automated health reports
- Perform documented disaster-recovery testing
