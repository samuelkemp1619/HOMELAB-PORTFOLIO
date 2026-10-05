# OPNsense & VLAN Network Architecture

## Overview

This project documents the redesign of my homelab network around OPNsense, VLAN segmentation, and a dedicated management network.

The goal was to move from a relatively flat home network to a segmented architecture that provides better security, management, monitoring, and room for future lab environments.

## Technologies

- OPNsense
- Proxmox VE
- VLANs / IEEE 802.1Q
- Managed Ethernet switching
- Linux bridges
- Firewall rules
- DHCP
- DNS
- Pi-hole
- Tailscale
- Prometheus / Grafana

## Architecture

OPNsense runs as a virtual machine within the Proxmox cluster and serves as the primary router and firewall for the homelab.

Proxmox uses a VLAN-aware Linux bridge to transport tagged network traffic between virtualized workloads and the physical network.

A managed switch provides VLAN connectivity between the physical Proxmox nodes and other network devices.

## Network Segmentation

The network includes separate logical networks for different trust and management requirements.

### Management Network

A dedicated management VLAN provides connectivity for infrastructure such as:

- Proxmox hosts
- OPNsense administration
- Monitoring infrastructure
- Other management services

### Trusted Network

A separate trusted VLAN provides connectivity for trusted client devices while keeping infrastructure management logically separated.

Additional VLANs are reserved or planned for workloads such as:

- Servers and self-hosted services
- IoT devices
- Security testing
- Lab environments
- Guest/untrusted devices

## Firewalling

OPNsense provides inter-VLAN routing and firewall enforcement.

The design follows the principle that communication between network segments should be explicitly permitted when required rather than allowing unrestricted communication between all VLANs.

This provides a foundation for increasingly granular access-control policies as additional services and security labs are deployed.

## DNS

Pi-hole provides network DNS filtering.

The DNS architecture allows centralized filtering and visibility while OPNsense remains responsible for routing and firewall enforcement.

Future improvements include additional DNS resiliency and monitoring.

## Remote Administration

Tailscale provides secure remote access to the homelab.

A subnet router allows authorized remote devices to reach selected internal networks without exposing administrative services directly to the public Internet.

Access-control policies and network segmentation remain part of the remote-access security model.

## Monitoring

Prometheus and Grafana provide infrastructure monitoring.

Network monitoring is being expanded to include:

- Gateway availability
- Service availability
- DNS health
- Network latency
- Packet loss
- Interface status
- Abnormal resource conditions
- Centralized log analysis

## Resiliency

The Proxmox cluster contains additional OPNsense virtual machines intended for disaster recovery and future failover capabilities.

The current environment does not assume automatic router failover. Maintenance involving the active router is therefore planned carefully to prevent unexpected network outages.

## Security Considerations

The architecture is designed around:

- Network segmentation
- Least-privilege firewall access
- Dedicated infrastructure management
- Reduced public exposure
- DNS filtering
- Secure remote administration
- Centralized monitoring
- Logging and future anomaly detection

## Troubleshooting & Validation

Implementation involved validating:

- VLAN tagging and untagging
- Switch port configuration
- Proxmox VLAN-aware bridges
- DHCP operation
- Gateway connectivity
- DNS resolution
- Inter-VLAN routing
- Firewall policy
- Cluster connectivity
- Remote administration

Changes were performed incrementally to minimize downtime and maintain administrative access during the network migration.

## Skills Demonstrated

- Network architecture
- VLAN design
- OPNsense administration
- Firewall configuration
- Layer 2 switching
- Layer 3 routing
- DHCP and DNS
- Linux networking
- Virtual networking
- Network segmentation
- Secure remote access
- Network troubleshooting
- Change management

## Future Improvements

- Expand VLAN segmentation
- Implement dedicated IoT and security-lab networks
- Improve firewall policy documentation
- Add network diagrams
- Expand network telemetry
- Centralize firewall and system logs
- Test OPNsense disaster recovery
- Evaluate automated gateway failover
- Implement exception-based health reporting
