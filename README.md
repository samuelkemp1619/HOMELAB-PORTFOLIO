# Network & Cybersecurity Homelab Portfolio

A practical portfolio of **network administration, systems engineering, cybersecurity, virtualization** projects. This lab provides hands-on experience designing segmented networks, operating virtualized services, validating controls, monitoring infrastructure, and documenting technical outcomes.

> **Documentation approach:** This portfolio showcases implemented projects and completed technical work. Validation limitations are documented explicitly, and unfinished projects are excluded from the featured project list. Network addresses, device-specific identifiers, and credentials are omitted or masked in public examples.

## Homelab Architecture Overview

![Illustrative four-node Proxmox homelab architecture](Homelab%20Architecture_%20Proxmox%20Cluster%20Overview.png)

*Illustrative architecture overview of the four-node Proxmox VE environment, including hosted services, monitoring, network segmentation, remote access, and backup workflows. Logical connections and some hardware/service annotations are schematic and should not be treated as independently verified topology or configuration.*

[View full-size architecture diagram](Homelab%20Architecture_%20Proxmox%20Cluster%20Overview.png)

## Featured Projects

| Project | Demonstrated skills | Status |
|---|---|---|
| [Windows Server 2025 Active Directory lab](virtualization/proxmox/case-studies/windows-active-directory-lab.md) | AD DS, Windows DNS, Windows 11 domain join, standard-user authentication | Authentication validated; GPO paused |
| [Four-node Proxmox VE cluster](virtualization/proxmox/) | Linux administration, KVM/QEMU, LXC, cluster operations, backup scheduling, completed November 2026 cluster update | Completed |
| [OPNsense and VLAN architecture](network/opnsense-vlan-architecture/) | VLAN segmentation, routing, firewall configuration, DNS policy | Completed |
| [Unrestricted wireless VLAN 50](network/opnsense-vlan-architecture/case-studies/unrestricted-wireless-vlan50.md) | 802.1Q wireless segmentation, OPNsense DHCP/firewall, packet-capture troubleshooting | Completed |
| [Redundant Pi-hole DNS](network/pihole-dns/) | DNS filtering, dual resolvers, DHCP integration, selected recovery testing | Completed |
| [Prometheus and Grafana monitoring](monitoring/prometheus-grafana/) | Metrics collection, dashboards, service monitoring | Completed |
| [Network architecture diagrams](network/diagrams/) | Logical and physical topology documentation | Completed |
| [Custom three-tier micro-PC holder](hardware/micro-pc-rack/) | CAD iteration, 3D printing, hardware integration | Completed |
| [Rack shelf and node installation](hardware/rack-shelf-installation/) | Rack mounting, cabling, physical infrastructure | Completed |
| [Graphy Grafana kiosk](monitoring/graphy-grafana-kiosk/) | Raspberry Pi, Linux, kiosk deployment, dashboard presentation | Completed |
| [Vaultwarden password audit](security/vaultwarden-password-audit/) | PowerShell, security hygiene analysis, reporting; historical audit findings | Completed |
| [Homelab credential rotation and MFA rollout](security/vaultwarden-password-audit/#november-2026-homelab-credential-hardening) | Credential rotation, TOTP enrollment, authentication testing, recovery planning | Completed |

## Featured Technical Case Studies

- [Four-Node Proxmox Cluster Operations and Recovery](virtualization/proxmox/case-studies/four-node-cluster-operations.md)
- [Power-Loss Recovery and Troubleshooting Incident](virtualization/proxmox/case-studies/power-loss-recovery-incident.md)
- [Unrestricted Wireless VLAN 50 — Deployment and DHCP Troubleshooting](network/opnsense-vlan-architecture/case-studies/unrestricted-wireless-vlan50.md)
- [Redundant Pi-hole DNS and Failover Testing](network/pihole-dns/case-studies/dns-redundancy-and-failover.md)

- [Windows Server 2025 Active Directory Deployment and Domain Authentication](virtualization/proxmox/case-studies/windows-active-directory-lab.md)

## Selected Accomplishments

### Segmented network and DNS resilience

- Configured VLAN-based separation for management, trusted, IoT, guest, and a dedicated unrestricted-internet wireless segment (VLAN 50), with OPNsense routing and firewall policies.
- Deployed a Grandstream SSID on VLAN 50 using Cloudflare DNS instead of Pi-hole; resolved a Dnsmasq interface-selection DHCP failure using multi-point packet captures. Client internet access and internal-service isolation were tested; subsequent validation was confirmed complete by the operator.
- Deployed two Pi-hole instances on separate Proxmox hosts and configured DHCP to advertise both DNS resolvers.
- Configured hourly primary-to-secondary DNS configuration synchronization using Nebula Sync.
- Verified independent DNS resolution and filtering, including a controlled Linux resolver fallback test.


### Virtualization and infrastructure

- Operate a four-node Proxmox VE environment with Linux containers and virtual machines.
- Completed the November 5, 2026 cluster maintenance campaign: all four nodes updated to Proxmox Manager 9.2.25 and kernel 7.0.14-28-pve, rebooted and verified quorate; OPNsense and Pi-hole confirmed running.
- Configured scheduled backups with retention policies and failure notifications.
- Designed, printed, and assembled a three-tier micro-PC holder and integrated it with rack shelves and cable management.

### Windows domain services and authentication

- Deployed a Windows Server 2025 Active Directory domain controller with integrated DNS, joined a Windows 11 Pro test workstation, and verified standard-user domain authentication through the VM console.
- Documented DNS registration and virtual network troubleshooting; Group Policy testing remains paused.

### Monitoring and security engineering

- Deployed Prometheus/Grafana infrastructure dashboards and a Raspberry Pi-based Grafana kiosk.
- Built and completed a PowerShell-based Vaultwarden audit covering **573 login entries**, producing aggregate security-hygiene findings.
- Completed a separate November 2026 homelab credential-hardening effort: rotated administrator passwords to unique values across the four Proxmox nodes and other homelab systems, as reported by the operator. Verified `root@pam` TOTP login through the Proxmox cluster, tested OPNsense local TOTP authentication and web login, and confirmed Vaultwarden two-step login and independently stored recovery information. The OPNsense login test required a TOTP-code-first credential format.
- Updated CubeCoders AMP management tools and the running ADS instance to version 2.8.0.10, corrected the container's UTF-8 locale, enabled its scheduled task timer, and verified the web interface returned HTTP 200.

## Infrastructure at a Glance

| Public alias | Hardware platform | Primary role |
|---|---|---|
| Node A | Dell OptiPlex 3000; i5-12500T (6C/12T), 32 GB DDR4-3200, 1 TB Samsung 970 EVO Plus NVMe | Proxmox infrastructure and virtual firewall services |
| Node B | HP Elite Mini 600 G9; Intel Core i5-12500T (6C/12T), 32 GB DDR5, 512 GB Samsung PM9A1 NVMe | Applications, monitoring, and backup storage |
| Node C | HP Elite Mini 600 G9; Intel Core i5-12500T (6C/12T), 16 GB DDR5, 500 GB Samsung 980 NVMe | Containers and secondary DNS services |
| Node D | Dell PowerEdge R640; dual Intel Xeon Silver 4110 (8C/16T each; 16C/32T total) | Windows Server lab and lab workloads |

These aliases are used for public presentation rather than exposing internal hostnames. Node A's Dell OptiPlex 3000 model, processor, RAM, storage and Ethernet controller were verified directly on the host on November 5, 2026. Node B–D specifications reflect previously recorded hardware details; Node D memory and storage are omitted because their capacities have not been confirmed for this summary.

**Technologies:** Proxmox VE · Debian/Linux · KVM/QEMU · LXC · OPNsense · VLANs · Pi-hole · Nebula Sync · Prometheus · Grafana · Tailscale · PowerShell · Python · Docker · Raspberry Pi

## Project Documentation

Each linked project page provides implementation context and, where available, architecture, testing, troubleshooting, security considerations, and lessons learned. The [network diagrams](network/diagrams/) distinguish logical relationships from verified physical connections.

## Public Documentation and Security

Public materials avoid exposing full internal host IP addresses, device MAC addresses, private hostnames, email addresses, credentials, tokens, serial numbers, and other unique infrastructure identifiers. Where network examples are needed, host octets are masked (for example, `192.168.20.XXX`).

Previously published information may remain in Git history; this README update does not rewrite historical commits.

---

**Focus areas:** Network Administration · Systems Administration · Cybersecurity · Virtualization · Linux · Monitoring · Troubleshooting · Automation
