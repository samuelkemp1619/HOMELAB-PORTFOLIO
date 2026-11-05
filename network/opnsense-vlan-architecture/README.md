# OPNsense & VLAN Network Architecture

**Status:** Deployed network segmentation and VLAN 50 wireless deployment.

## Overview

This project documents the redesign of my homelab network around OPNsense, VLAN segmentation, and a dedicated management network.

The goal was to move from a relatively flat home network to a segmented architecture that provides better security, management, monitoring, and separation between trust zones.

## Completed deployment: Unrestricted internet Wi-Fi (VLAN 50)

A dedicated Grandstream GWN7660E SSID was deployed on VLAN 50 through the TP-Link managed switch and Proxmox VLAN-aware bridge to OPNsense VM 100. OPNsense Dnsmasq provides DHCP and advertises Cloudflare DNS instead of Pi-hole; ordered IPv4 rules allow internet traffic while blocking access to the firewall and defined private networks.

Client connectivity, Cloudflare DNS diagnostic results, and inability to reach internal services were tested. A DHCP outage was traced with physical, bridge, and VM-interface packet captures to Dnsmasq not listening on the new interface; adding UNSECURE resolved it. The operator subsequently confirmed validation complete for this deployment. Detailed IPv6 and per-application test evidence was not supplied for publication.

[Read the deployment and troubleshooting case study](case-studies/unrestricted-wireless-vlan50.md).

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

## Verified Wireless SSID Segmentation

On November 5, 2026, the Grandstream GWN7660E SSID configuration and TP-Link TL-SG108E switch port 5 VLAN membership were checked together.

| Wireless network (public alias) | VLAN | AP/switch tagging |
|---|---|---|
| Primary Wi-Fi | 10 (management) | Untagged; switch port 5 PVID 10 |
| IoT Wi-Fi | 30 | Tagged |
| Guest Wi-Fi | 40 | Tagged |
| Unrestricted Wi-Fi | 50 | Tagged |

The primary Wi-Fi network uses the management VLAN rather than VLAN 20 (trusted). VLAN 20 is not a member of the AP's switch port. This records the configured mapping without asserting that management-service access has been tested from wireless clients.

## Firewalling

OPNsense provides inter-VLAN routing and firewall enforcement.

The design follows the principle that communication between network segments should be explicitly permitted when required rather than allowing unrestricted communication between all VLANs.

This provides a foundation for access-control policies between network segments.

## DNS

Pi-hole provides network DNS filtering.

The DNS architecture allows centralized filtering and visibility while OPNsense remains responsible for routing and firewall enforcement.



## Remote Administration

Tailscale provides secure remote access to the homelab.

A subnet router allows authorized remote devices to reach selected internal networks without exposing administrative services directly to the public Internet.

Access-control policies and network segmentation remain part of the remote-access security model.

## Monitoring

Prometheus and Grafana provide infrastructure monitoring.

## Resiliency

The active OPNsense VM provides routing and firewall services.

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
- Logging

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

