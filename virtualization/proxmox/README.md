# Four-Node Proxmox VE Cluster

## Project Overview

Designed, deployed, and administer a four-node Proxmox VE environment supporting virtualized networking, DNS filtering, monitoring, password management, Windows Server experimentation, and automation. The project demonstrates practical Linux systems administration, virtualization operations, network segmentation, service recovery, and backup management.

**Status:** Operational cluster. Some resilience and disaster-recovery improvements remain planned or paused.

## Architecture

| Public node alias | Hardware | Workload focus |
|---|---|---|
| Node A | Dell OptiPlex 3000 (SMBIOS-reported) | Virtual firewall and DNS infrastructure |
| Node B | HP Elite Mini 600 G9 | Application hosting and monitoring |
| Node C | HP Elite Mini 600 G9 | DNS, password management, and automation containers |
| Node D | Dell PowerEdge R640 | Windows Server lab and AI-Operations virtual machine |

Node aliases intentionally replace internal hostnames in public documentation. The exact chassis designation of Node A has not been independently confirmed.

### Core technologies

**Proxmox VE · Debian Linux · KVM/QEMU · LXC · Corosync · LVM-thin · NFS · Linux bridges · 802.1Q VLANs · ZSTD · Prometheus · Grafana**

The four-node cluster uses Corosync for membership and quorum. With four expected votes and standard majority quorum, three votes are required. **Quorum is not the same as automatic VM failover**; no unverified high-availability behavior is claimed here.

## Service Placement

| Node | Documented workloads |
|---|---|
| Node A | OPNsense virtual firewall; primary Pi-hole DNS container |
| Node B | AMP game-server management; Dashy, Prometheus, Grafana and exporters |
| Node C | Vaultwarden; secondary Pi-hole DNS; Python-based Gmail organization container |
| Node D | Windows Server 2025 VM; AI-Operations VM with Docker-based services |

Workload names are included for technical context; internal host addresses, MAC addresses, and VM management endpoints are omitted.

## Network Design

- Management and trusted networks are separated using VLANs; IoT, guest, and WAN VLANs are also configured in the lab.
- Nodes A, B, and C use VLAN-aware Linux bridges and a tagged management interface.
- Node D currently has a management bridge that is **not configured as VLAN-aware**; the network should not be described as uniformly VLAN-aware.
- OPNsense provides virtualized routing and firewall policy. Physical switch-port assignments have not been fully verified.

See the [network architecture documentation](../../network/diagrams/) for diagrams with identifying network details masked.

## Operational Work and Validation

### Cluster administration

- Confirmed four-node cluster quorum during October 2026 maintenance.
- Migrated the monitoring/application container to Node B and verified dashboard and network access afterward.
- Managed virtual machines, Linux containers, storage volumes, networking, and service startup behavior.
- Completed a controlled power-loss simulation and investigated subsequent network, quorum, and DNS recovery issues. Recovery work restored DNS resolution and routing; the test should **not** be presented as evidence of seamless high availability.

### DNS and service resilience

- Operate primary and secondary Pi-hole containers on different Proxmox hosts.
- Validated independent DNS resolution and filtering, with a controlled Linux resolver fallback test.
- Other client-side failover behavior remains unverified.

### Backups

- Configured scheduled Proxmox `vzdump` backups to off-node storage using ZSTD compression.
- Documented a snapshot-mode job and a separate stop-mode job.
- Applied retention targets of **3 latest, 7 daily, 4 weekly, and 3 monthly** backups.
- Reviewed successful backup-task logs and failure notification configuration.

**Validation boundary:** Successful backup creation does not establish recoverability. A complete documented restore test remains pending; backup coverage should not be described as comprehensive.

### Windows and automation

- Installed and activated a Windows Server 2025 lab VM; further directory-services configuration is paused.
- Deployed an AI-Operations VM hosting Docker, PostgreSQL, and n8n.
- Configured and manually tested monitoring alert-routing logic; the original scheduled workflow remains **unpublished and paused**. This is not a claim of active scheduled alert delivery.

## Challenges and Lessons Learned

| Challenge | Response / outcome |
|---|---|
| Cluster behavior following simulated power loss | Investigated quorum and network reachability, restored essential DNS and routing, and identified recovery dependencies |
| Monitoring workload placement | Migrated the application/monitoring container and verified service accessibility afterward |
| DNS availability during host disruption | Maintained a secondary resolver on another node and validated selected recovery scenarios |
| Backup reliability | Established scheduled jobs, retention, and log verification while distinguishing backup success from restore readiness |
| Mixed host network configurations | Documented differences between VLAN-aware node bridges and the remaining host's untagged management bridge |

## Skills Demonstrated

- Proxmox VE, KVM/QEMU, and LXC administration
- Linux networking, bridges, VLANs, and virtual firewall integration
- Cluster membership and quorum troubleshooting
- Storage and backup policy management
- VM/container migration and operational validation
- Monitoring with Prometheus and Grafana
- Recovery troubleshooting and evidence-based technical documentation

## Planned / Paused Work

- Documented restore and disaster-recovery testing
- Expanded backup coverage and alerting
- Centralized log collection and exception-based reporting
- Further Windows Server / Active Directory lab configuration
- Additional security monitoring and network validation

## Security and Documentation Scope

This public write-up uses generic node aliases and omits full internal IP addresses, device MAC addresses, unique device identifiers, credentials, and private endpoints. Historical Git commits are not rewritten by this documentation update.

**Related projects:** [OPNsense and VLANs](../../network/opnsense-vlan-architecture/) · [Redundant Pi-hole](../../network/pihole-dns/) · [Monitoring](../../monitoring/prometheus-grafana/) · [Physical network diagrams](../../network/diagrams/)
