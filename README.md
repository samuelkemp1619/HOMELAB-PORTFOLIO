# Homelab Portfolio

A hands-on Network & Systems Administration and Cybersecurity homelab built to develop and demonstrate practical experience with enterprise networking, virtualization, Linux administration, monitoring, security, automation, and self-hosted infrastructure.

## 📚 Project Documentation

Explore the major projects in this homelab:

| Project | Description |
|---------|-------------|
| [Proxmox VE Cluster](virtualization/proxmox/) | Four-node virtualization cluster using Proxmox VE, KVM/QEMU, LXC, Corosync, shared infrastructure, and backup management. |
| [OPNsense & VLAN Architecture](network/opnsense-vlan-architecture/) | Segmented network architecture using OPNsense, VLANs, firewall policies, and managed switching. |
| [Pi-hole DNS](network/pihole-dns/) | Redundant two-instance Pi-hole v6 DNS filtering deployment, with OPNsense DHCP distribution and hourly primary-to-secondary configuration synchronization using Nebula Sync. Automatic client failover remains under validation. |
| [Prometheus & Grafana Monitoring](monitoring/prometheus-grafana/) | Centralized metrics, dashboards, infrastructure monitoring, and planned exception-based reporting. |
| [Network Architecture](network/diagrams/) | High-level diagrams documenting the physical, virtual, and network architecture of the homelab. |
| [Three-Tier Proxmox Micro-PC Rack](hardware/micro-pc-rack/) | Completed custom 3D-printed three-node rack holder, M4 assembly, and physical infrastructure integration. |
| [Rack Shelf & Node Installation](hardware/rack-shelf-installation/) | Completed 12-24 rack hardware compatibility troubleshooting, shelf installation, and three-node physical integration. |
| [Graphy — Grafana Monitoring Kiosk](monitoring/graphy-grafana-kiosk/) | Completed Raspberry Pi 4 Chromium/labwc Grafana playlist kiosk on trusted VLAN 20. |
| [AI-Operations Proxmox Monitoring](automation/ai-operations-proxmox-monitoring/) | Four-node monitoring, AI health reporting, and tested alert routing; Pushover delivery validated, scheduled activation pending. |
| [Vaultwarden Password Audit](security/vaultwarden-password-audit/) | Completed PowerShell security hygiene audit of 573 login entries with aggregate findings and safe reporting. |

### Completed Project Highlights

- **Redundant Pi-hole DNS:** Deployed primary and secondary Pi-hole v6.7.1 instances on separate Proxmox hosts, configured OPNsense DHCP to advertise both DNS addresses across four networks, and deployed hourly Nebula Sync from the primary to the secondary. Both instances independently pass DNS resolution and blocking tests. Automatic client failover remains unverified.
- **Physical rack integration:** Three-node micro-PC holder, rack shelves, M4 hardware, and installation — completed October 2026.
- **Graphy monitoring kiosk:** Raspberry Pi 4 displaying rotating Grafana dashboards — completed October 3, 2026.
- **Vaultwarden password audit:** PowerShell script and CSV analysis of 573 login entries — completed October 2026. The audit identified issues; it did not automatically remediate passwords.

### Paused / Future Work

The following are **paused**, not implemented or completed portfolio claims:

- Exception-based homelab health reporting and Grafana Loki centralized logging
- Backup and disaster-recovery improvements
- Windows Server / Active Directory lab
- Expanded cybersecurity monitoring and detection
- Additional network segmentation, redundancy, and infrastructure improvements
- Other self-hosted services under consideration

Removed projects (including RFID and Raspberry Pi hardware-security experiments) are not part of the current development backlog.

## 🖥️ Current Infrastructure

My homelab is built around a four-node Proxmox VE cluster:

| Node | Hardware | Role |
|------|----------|------|
| HomeProx | Dell OptiPlex 7020 Micro | Proxmox VE / Primary Infrastructure |
| HomeProx2 | HP Elite Mini 600 G9 | Proxmox VE / Application Host |
| HomeProx3 | HP Elite Mini 600 G9 | Proxmox VE / Container Host |
| HomeProx4 | Dell PowerEdge R640 | Proxmox VE / Windows Server and AI-Operations Host |

### Core Technologies

- Proxmox VE
- OPNsense
- VLAN segmentation
- Redundant Pi-hole DNS and Nebula Sync
- Prometheus
- Grafana
- Tailscale
- Linux containers (LXC)
- Virtual machines
- Network monitoring
- Automated backups

## 🌐 Networking

The network uses OPNsense for routing and firewalling with multiple VLANs separating infrastructure and trusted devices.

Projects include:

- VLAN design and segmentation
- Firewall policy
- DNS filtering and redundancy
- Remote administration
- Network monitoring
- Proxmox cluster networking
- Secure management network

## 📊 Monitoring

Monitoring is provided through Prometheus and Grafana.

Current and planned monitoring projects include:

- Proxmox node monitoring
- VM and container monitoring
- Network health monitoring
- Grafana dashboards
- Centralized logging
- Exception-based health reporting
- Automated email health reports

## 🔐 Cybersecurity

Security projects include or are planned around:

- Network segmentation
- Firewall policy
- Centralized logging
- IDS/IPS
- SIEM
- Security monitoring and alerting

All offensive/security-testing projects are performed in controlled lab environments against systems I own or am authorized to test.

## 🛠️ Infrastructure Projects

This repository will document projects including:

- Four-node Proxmox cluster
- OPNsense virtual firewall/router
- [Pi-hole DNS filtering and redundancy](network/pihole-dns/)
- Prometheus and Grafana monitoring
- Tailscale remote administration
- Automated Proxmox backups
- Homelab health reporting
- Windows Server / Active Directory lab
- Self-hosted services
- [Completed three-tier 3D-printed rack hardware](hardware/micro-pc-rack/)
- [Completed rack shelf and node installation](hardware/rack-shelf-installation/)
- [Completed Graphy monitoring kiosk](monitoring/graphy-grafana-kiosk/)
- [AI-Operations monitoring and alert automation](automation/ai-operations-proxmox-monitoring/)
- [Completed Vaultwarden PowerShell password audit](security/vaultwarden-password-audit/)

## 📁 Repository Structure

Completed project write-ups are organized under `hardware/`, `monitoring/`, and `security/`, alongside the existing `network/`, `virtualization/`, and `troubleshooting/` documentation.

Each project will have its own documentation covering:

- Project goals
- Architecture
- Implementation
- Configuration
- Troubleshooting
- Security considerations
- Testing and validation
- Skills demonstrated
- Lessons learned

## 🔒 Security & Privacy

Configuration files and examples published in this repository are sanitized before publication.

Passwords, API keys, authentication tokens, private keys, personal information, and other sensitive information are not intentionally included.

## 🎯 Purpose

This lab serves as both a learning environment and a portfolio demonstrating practical skills in:

**Network Administration • Systems Administration • Cybersecurity • Virtualization • Linux • Windows • Monitoring • Troubleshooting • Automation**
