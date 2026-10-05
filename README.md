# Homelab Portfolio

A hands-on Network & Systems Administration and Cybersecurity homelab built to develop and demonstrate practical experience with enterprise networking, virtualization, Linux administration, monitoring, security, automation, and self-hosted infrastructure.

## 📚 Project Documentation

Explore the major projects in this homelab:

| Project | Description |
|---------|-------------|
| [Proxmox VE Cluster](virtualization/proxmox/) | Three-node virtualization cluster using Proxmox VE, KVM/QEMU, LXC, Corosync, shared infrastructure, and backup management. |
| [OPNsense & VLAN Architecture](network/opnsense-vlan-architecture/) | Segmented network architecture using OPNsense, VLANs, firewall policies, and managed switching. |
| [Pi-hole DNS](network/pihole-dns/) | Centralized DNS filtering and network-wide DNS administration using Pi-hole. |
| [Prometheus & Grafana Monitoring](monitoring/prometheus-grafana/) | Centralized metrics, dashboards, infrastructure monitoring, and planned exception-based reporting. |
| [Network Architecture](network/diagrams/) | High-level diagrams documenting the physical, virtual, and network architecture of the homelab. |

### Projects in Development

Future documentation will cover:

- Automated exception-based homelab health reporting
- Grafana Loki centralized logging
- Backup and disaster recovery
- Tailscale secure remote administration
- Windows Server and Active Directory lab
- Cybersecurity monitoring and detection
- Raspberry Pi security projects
- IoT security lab
- RFID access-control and security testing
- Honeypot deployment
- 3D-printed micro-cluster rack mounting
- Self-hosted infrastructure and services

## 🖥️ Current Infrastructure

My homelab is built around a three-node Proxmox VE cluster:

| Node | Hardware | Role |
|------|----------|------|
| HomeProx | Dell OptiPlex 7020 Micro | Proxmox VE / Primary Infrastructure |
| HomeProx2 | HP Elite Mini 600 G9 | Proxmox VE / Application Host |
| HomeProx3 | HP Elite Mini 600 G9 | Proxmox VE / Monitoring Host |

### Core Technologies

- Proxmox VE
- OPNsense
- VLAN segmentation
- Pi-hole DNS
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
- DNS filtering
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
- Honeypots
- IoT security testing
- RFID access-control security
- Security monitoring and alerting

All offensive/security-testing projects are performed in controlled lab environments against systems I own or am authorized to test.

## 🛠️ Infrastructure Projects

This repository will document projects including:

- Three-node Proxmox cluster
- OPNsense virtual firewall/router
- Pi-hole DNS filtering
- Prometheus and Grafana monitoring
- Tailscale remote administration
- Automated Proxmox backups
- Homelab health reporting
- Windows Server / Active Directory lab
- Raspberry Pi security projects
- Self-hosted services
- 3D-printed rack hardware

## 📁 Repository Structure

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
