# Case Study: Four-Node Proxmox Cluster Operations and Recovery

**Domain:** Systems administration, virtualization, network engineering, resilience  
**Platform:** Proxmox VE 9.x on Debian 13  
**Scope:** Four physical hosts, KVM virtual machines, LXC containers, VLAN networking, monitoring, and scheduled backups  
**Evidence window:** October 2026 operational and maintenance records

## Executive Summary

Designed and operated a four-node Proxmox VE cluster combining three compact systems and one rack server. The lab hosts firewall, DNS, monitoring, password-management, automation, and Windows Server workloads. The work emphasizes operational discipline: documenting host roles, validating cluster quorum, migrating services, testing power-loss recovery, and reviewing backup jobs.

The environment is operational, but the power-loss exercise exposed dependencies between host availability, cluster membership, networking, and DNS. Recovery required intervention. This case study does **not** represent the cluster as a validated high-availability deployment.

## Objectives and Constraints

- Consolidate mixed workloads onto a manageable virtualization platform.
- Segment management and client networks while supporting virtualized firewall services.
- Keep critical DNS functions available across separate hosts.
- Collect operational metrics and expose service health through dashboards.
- Schedule compressed backups with retention and failure notifications.
- Document observed outcomes without conflating configured features with tested resilience.

## Architecture and Design Choices

| Public alias | Hardware | Documented role |
|---|---|---|
| Node A | Dell OptiPlex 3000 Micro (SMBIOS-reported designation) | Virtual firewall and primary DNS infrastructure |
| Node B | HP Elite Mini 600 G9 | Monitoring and application services |
| Node C | HP Elite Mini 600 G9 | Secondary DNS, password management, utility containers |
| Node D | Dell PowerEdge R640 | Windows Server lab and AI-Operations VM |

**Why mixed hardware?** Compact nodes provide a practical platform for always-on infrastructure and service isolation, while the rack server supports more resource-intensive lab workloads. The trade-off is nonuniform storage, networking, and host characteristics, which require explicit documentation rather than assumptions of interchangeability.

**Why both VMs and LXCs?** VMs support full guest operating systems and virtual firewall workloads; containers offer lighter-weight isolation for Linux-based applications. Placement reflects workload requirements and operational convenience rather than an asserted HA placement policy.

**Cluster behavior:** Corosync manages membership and quorum. In a four-vote configuration, three votes are required for majority quorum. Quorum allows coordinated cluster operations; it does not by itself restart guests on surviving nodes.

![Illustrative Proxmox architecture overview](../../../Homelab%20Architecture_%20Proxmox%20Cluster%20Overview.png)

*Illustrative overview, not a verified physical cabling map. Exact VLAN paths and some hardware annotations are schematic.*

## Network and Security Design

Management and trusted traffic use separate VLANs, with additional lab, IoT, guest, and WAN-related VLAN configurations. OPNsense supplies virtualized routing and firewall policy. Linux bridges provide guest connectivity; network configuration is **not identical across all four hosts**. In particular, Node D's management bridge was documented as not VLAN-aware, and complete physical switch-port mapping was not independently validated.

For public documentation, hostnames and endpoint identifiers are replaced by generic aliases. The architecture illustration is schematic and should not be used as a configuration source of truth.

## Implementation and Operational Validation

### 1. Establish cluster membership and workload placement

- Administered a four-node Proxmox cluster and verified quorum during October maintenance.
- Used KVM/QEMU virtual machines and LXC containers for different workload classes.
- Migrated the monitoring/application container to Node B, then verified service reachability and dashboards.
- Maintained Prometheus, Grafana, Dashy, and exporter-based observability.

**Observed outcome:** The monitoring stack remained accessible following its documented migration. This does not prove that every workload supports live migration or automatic recovery.

### 2. Test power-loss recovery

A controlled full-power-off simulation was performed in October 2026. After power was restored, two compact nodes came back online while another experienced missing network link indications and quorum-related problems. Virtualized firewall connectivity was also disrupted.

**Troubleshooting sequence and findings:**

1. Checked which hosts and network paths returned after power restoration.
2. Investigated host link state, cluster membership/quorum, and virtual firewall reachability.
3. Re-established DNS service using a Pi-hole container on a surviving host.
4. Verified name resolution and routing after correcting client-side DNS behavior.
5. Isolated a desktop Tailscale DNS-override conflict; disabling that override restored expected Pi-hole resolution.

**Observed outcome:** DNS and routing were restored through troubleshooting and configuration changes. Recovery was not seamless and should not be described as successful automatic failover.

**Engineering lesson:** Cluster quorum, physical network availability, virtual firewall startup, DNS resolution, and remote-access configuration form distinct recovery dependencies. Test each separately rather than treating “host powered on” as equivalent to “service recovered.”

### 3. Validate DNS resilience within a defined scope

Two Pi-hole instances were deployed on separate hosts, with configuration synchronization and more than one resolver available to clients. Independent DNS resolution and filtering were verified, along with a controlled Linux resolver fallback scenario.

**Limit:** Automatic fallback behavior across every client operating system and failure mode has not been established.

### 4. Configure and review backups

Two scheduled Proxmox backup jobs were documented: snapshot mode at **03:00** and stop mode at **03:30**. The configuration uses ZSTD compression, off-node backup storage, failure notifications, and pruning targets of **3 latest / 7 daily / 4 weekly / 3 monthly**.

Task logs confirmed successful backup execution for reviewed jobs. However, the dashboard has shown guests without backup coverage, and a complete restore exercise has not been documented.

**Limit:** Verified job completion is not equivalent to complete guest coverage or recoverability. Restore testing and coverage remediation remain open work.

## Challenges, Responses, and Evidence Boundaries

| Challenge | Response | What remains unverified |
|---|---|---|
| Loss of connectivity after power restoration | Traced host, quorum, firewall, and DNS dependencies; restored essential services | Unattended recovery and guest failover |
| Monitoring service relocation | Migrated monitoring container and checked access afterward | HA restart and universal workload mobility |
| DNS availability | Operated resolvers on separate hosts and tested selected fallback behavior | All-client automatic failover |
| Backup consistency | Scheduled compressed jobs, retention, notifications, and log checks | Complete guest coverage and successful restore |
| Nonuniform host networking | Documented bridge/VLAN differences and avoided assuming uniform configuration | Full physical switch-port and VLAN-path validation |

## Skills Demonstrated

- Proxmox VE cluster administration and Corosync quorum analysis
- KVM/QEMU and LXC workload placement and migration
- Linux bridge and VLAN configuration awareness
- Virtual firewall and DNS dependency troubleshooting
- Observability with Prometheus, Grafana, and exporters
- Backup scheduling, retention, and evidence-based validation
- Incident documentation, recovery sequencing, and operational risk assessment

## Next Engineering Milestones

1. Inventory all guests against backup-job inclusion and close documented coverage gaps.
2. Perform and record an isolated guest restore test, including time-to-restore and application validation.
3. Document power-on dependencies for routing, DNS, and management access.
4. Verify switch-port VLAN configuration and host bridge behavior.
5. Define and test an explicit recovery procedure before making any HA claims.

## Related Documentation

- [Proxmox cluster project overview](../README.md)
- [Network architecture](../../../network/diagrams/)
- [OPNsense and VLANs](../../../network/opnsense-vlan-architecture/)
- [Pi-hole DNS](../../../network/pihole-dns/)
- [Prometheus and Grafana](../../../monitoring/prometheus-grafana/)

*Security note: Public node aliases replace internal hostnames. No credentials, full host IP addresses, MAC addresses, serial numbers, or private management endpoints are included.*
