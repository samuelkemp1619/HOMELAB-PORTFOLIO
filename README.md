# Homelab Portfolio

A hands-on Network & Systems Administration and Cybersecurity homelab built to develop and demonstrate practical experience with enterprise networking, virtualization, Linux administration, monitoring, security, automation, and self-hosted infrastructure.

## 📚 Project Documentation

Explore the major projects in this homelab:

| Project | Description |
|---------|-------------|
| [Proxmox VE Cluster](virtualization/proxmox/) | Four-node virtualization cluster using Proxmox VE, KVM/QEMU, LXC, Corosync, shared infrastructure, and backup management. |
| [OPNsense & VLAN Architecture](network/opnsense-vlan-architecture/) | Segmented network architecture using OPNsense, VLANs, firewall policies, managed switching, and staged DNS-bypass enforcement. |
| [Pi-hole DNS](network/pihole-dns/) | Redundant two-instance Pi-hole v6 DNS filtering deployment, with OPNsense DHCP distribution and hourly primary-to-secondary configuration synchronization using Nebula Sync. DNS-bypass rules are configured across four networks; client validation and DoH controls remain outstanding. Linux resolver failover passed a controlled isolated test; Windows and other client failover remain unverified. |
| [Prometheus & Grafana Monitoring](monitoring/prometheus-grafana/) | Centralized metrics, dashboards, infrastructure monitoring, and planned exception-based reporting. |
| [Network Architecture](network/diagrams/) | High-level diagrams documenting the physical, virtual, and network architecture of the homelab. |
| [Three-Tier Proxmox Micro-PC Rack](hardware/micro-pc-rack/) | Completed custom 3D-printed three-node rack holder, M4 assembly, and physical infrastructure integration. |
| [Rack Shelf & Node Installation](hardware/rack-shelf-installation/) | Completed 12-24 rack hardware compatibility troubleshooting, shelf installation, and three-node physical integration. |
| [Graphy — Grafana Monitoring Kiosk](monitoring/graphy-grafana-kiosk/) | Completed Raspberry Pi 4 Chromium/labwc Grafana playlist kiosk on trusted VLAN 20. |
| [AI-Operations Proxmox Monitoring](automation/ai-operations-proxmox-monitoring/) | Four-node monitoring, AI health reporting and tested alert routing; scheduled workflow remains unpublished. |
| [Vaultwarden Password Audit](security/vaultwarden-password-audit/) | Completed PowerShell security hygiene audit of 573 login entries with aggregate findings and safe reporting. |

### Completed Project Highlights

- **Redundant Pi-hole DNS:** Deployed primary and secondary Pi-hole v6.7.1 instances on separate Proxmox hosts, configured OPNsense DHCP to advertise both DNS addresses across four networks, and deployed hourly Nebula Sync from the primary to the secondary. Both instances independently pass DNS resolution and blocking tests. Linux resolver failover passed; other client behavior remains unverified.
- **DNS bypass enforcement — configuration phase:** Added ordered OPNsense rules on LAN, TRUSTED, IOT, and GUEST to permit DNS to both Pi-hole instances and block direct IPv4 DNS on port 53 and DNS-over-TLS on TCP port 853. LAN testing confirmed both Pi-holes resolve queries and a direct query to `8.8.8.8` times out. Enforcement from TRUSTED, IOT, and GUEST has not been client-tested; DNS-over-HTTPS protections are not configured or validated. This work is paused pending further testing.
- **Physical rack integration:** Three-node micro-PC holder, rack shelves, M4 hardware, and installation — completed October 2026.
- **Graphy monitoring kiosk:** Raspberry Pi 4 displaying rotating Grafana dashboards — completed October 3, 2026.
- **Vaultwarden password audit:** PowerShell script and CSV analysis of 573 login entries — completed October 2026. The audit identified issues; it did not automatically remediate passwords.

### Latest Network and Automation Validation (October 2026)

- **Pi-hole redundancy validation (October 30):** Both Pi-hole servers resolved public and internal DNS names; four Proxmox nodes list both resolvers; gravity domain sets (2,135,103 rows each) and enabled adlist configurations matched. An isolated HomeProx resolver test using an unreachable first server (`192.0.2.1`) and secondary Pi-hole (`192.168.10.3`) resolved `example.com`. The mount-namespace test inadvertently changed the host's `/etc/resolv.conf`; the original primary/secondary configuration was restored and both direct DNS lookups and system resolution passed afterward. This verifies Linux resolver fallback under the tested conditions, not automatic failover for all DHCP clients.


- **DNS policy validation:** Verified direct DNS queries to an external resolver are blocked in the tested LAN path, and observed a failed TCP connection to `1.1.1.1:853` during DNS-over-TLS testing. A failed connection alone does not establish that every network and client is covered; additional validation remains paused.
- **AI-Operations workflow:** The hourly n8n schedule and alert-routing logic were configured and tested, but the original workflow remains unpublished. Pushover configuration and tests should not be confused with live scheduled alerting.
- **Infrastructure documentation:** Retained the completed physical rack integration and backup scheduling work. Projects awaiting additional implementation remain explicitly marked as paused.

### Project Tracking (October 30, 2026)

- **Physical integration:** The three-tier holder, rack shelves, installed nodes and associated cable management are complete.

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
