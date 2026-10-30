# Network & Cybersecurity Homelab Portfolio

A practical portfolio of **network administration, systems engineering, cybersecurity, virtualization, and automation** projects. This lab provides hands-on experience designing segmented networks, operating virtualized services, validating controls, monitoring infrastructure, and documenting technical outcomes.

> **Documentation approach:** Implemented features, validated behavior, and future work are distinguished throughout. Network addresses, device-specific identifiers, and credentials are omitted or masked in public examples.

## Featured Projects

| Project | Demonstrated skills | Status |
|---|---|---|
| [Four-node Proxmox VE cluster](virtualization/proxmox/) | Linux administration, KVM/QEMU, LXC, cluster operations, backup scheduling | Operational |
| [OPNsense and VLAN architecture](network/opnsense-vlan-architecture/) | Network segmentation, firewall policy, managed switching, DNS policy | Operational; additional policy validation pending |
| [Redundant Pi-hole DNS](network/pihole-dns/) | DNS filtering, service redundancy, DHCP integration, Nebula Sync, failure testing | Deployed; selected failover tests completed |
| [Prometheus and Grafana monitoring](monitoring/prometheus-grafana/) | Metrics collection, observability, dashboards, service monitoring | Operational; reporting enhancements planned |
| [Network architecture diagrams](network/diagrams/) | Logical and physical topology documentation, VLAN design | Documented; exact switch-port mapping unverified |
| [Custom three-tier micro-PC holder](hardware/micro-pc-rack/) | CAD iteration, 3D printing, hardware integration | Completed |
| [Rack shelf and node installation](hardware/rack-shelf-installation/) | Rack mounting, cabling, physical infrastructure | Completed |
| [Graphy Grafana kiosk](monitoring/graphy-grafana-kiosk/) | Raspberry Pi, Linux, kiosk deployment, dashboard presentation | Completed |
| [Vaultwarden password audit](security/vaultwarden-password-audit/) | PowerShell, security hygiene analysis, reporting | Audit completed; findings not automatically remediated |
| [AI-Operations monitoring prototype](automation/ai-operations-proxmox-monitoring/) | Workflow design, alert classification, notification testing | Prototype tested; scheduled workflow unpublished and paused |

## Selected Accomplishments

### Segmented network and DNS resilience

- Configured VLAN-based separation for management, trusted, IoT, and guest traffic, with OPNsense routing and firewall policies.
- Deployed two Pi-hole instances on separate Proxmox hosts and configured DHCP to advertise both DNS resolvers.
- Configured hourly primary-to-secondary DNS configuration synchronization using Nebula Sync.
- Verified independent DNS resolution and filtering; a controlled Linux resolver fallback test succeeded. Automatic failover for every client type has **not** been established.
- Configured rules intended to block direct DNS and DNS-over-TLS bypass on four networks. LAN-path validation was performed; complete client coverage and DNS-over-HTTPS enforcement remain **unverified**.

### Virtualization and infrastructure

- Operate a four-node Proxmox VE environment with Linux containers and virtual machines.
- Maintain scheduled backups with retention policies and failure notifications; broader disaster-recovery validation remains future work.
- Designed, printed, and assembled a three-tier micro-PC holder and integrated it with rack shelves and cable management.

### Monitoring and security engineering

- Deployed Prometheus/Grafana infrastructure dashboards and a Raspberry Pi-based Grafana kiosk.
- Built and completed a PowerShell-based Vaultwarden audit covering **573 login entries**, producing aggregate security-hygiene findings without claiming password remediation.
- Developed and tested alert-routing logic for an AI-Operations monitoring workflow. Its production schedule is **not published**, and the project remains paused.

## Infrastructure at a Glance

| Public alias | Hardware platform | Primary role |
|---|---|---|
| Node A | Dell OptiPlex 3000 (SMBIOS-reported) | Proxmox infrastructure and virtual firewall services |
| Node B | HP Elite Mini 600 G9 | Applications and monitoring |
| Node C | HP Elite Mini 600 G9 | Containers and DNS services |
| Node D | Dell PowerEdge R640 | Windows Server lab and automation workloads |

These aliases are used for public presentation rather than exposing internal hostnames. The exact chassis designation of Node A is documented as SMBIOS-reported rather than independently confirmed.

**Technologies:** Proxmox VE · Debian/Linux · KVM/QEMU · LXC · OPNsense · VLANs · Pi-hole · Nebula Sync · Prometheus · Grafana · Tailscale · PowerShell · Python · Docker · PostgreSQL · Raspberry Pi

## Project Documentation

Each linked project page provides implementation context and, where available, architecture, testing, troubleshooting, security considerations, and lessons learned. The [network diagrams](network/diagrams/) distinguish logical relationships from verified physical connections.

## Current and Future Work

The following are **not presented as completed deployments**:

- Exception-based health reporting and centralized log analysis
- Expanded backup and disaster-recovery testing
- Additional Windows Server and Active Directory lab work
- Further network security validation and detection capabilities
- Publication of the paused AI-Operations scheduled alert workflow

## Public Documentation and Security

Public materials avoid exposing full internal host IP addresses, device MAC addresses, private hostnames, email addresses, credentials, tokens, serial numbers, and other unique infrastructure identifiers. Where network examples are needed, host octets are masked (for example, `192.168.20.XXX`).

Previously published information may remain in Git history; this README update does not rewrite historical commits.

---

**Focus areas:** Network Administration · Systems Administration · Cybersecurity · Virtualization · Linux · Monitoring · Troubleshooting · Automation
