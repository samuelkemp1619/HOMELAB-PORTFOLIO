# Incident Case Study: Power-Loss Recovery and DNS Troubleshooting

**Incident type:** Controlled power-failure simulation and subsequent recovery  
**Date:** October 22, 2026  
**Environment:** Four-node Proxmox VE 9.x cluster, virtualized firewall services, Pi-hole DNS, Tailscale clients  
**Outcome:** Essential DNS resolution and network access restored through manual troubleshooting  
**Classification:** Planned resilience test with unplanned recovery complications

## Executive Summary

A planned full-power-off exercise was used to evaluate how a mixed-hardware Proxmox homelab returned to service after loss of power. The test exposed multiple independent recovery dependencies: physical network link availability, Proxmox cluster quorum, virtual firewall reachability, DNS service placement, and client-side Tailscale DNS settings.

Following restoration of power, Nodes B and C returned to service, while Node A showed no Ethernet link lights and encountered quorum-related errors. OPNsense virtual machines were not reachable at that stage. DNS was re-established through a Pi-hole container on Node C. Later, a desktop DNS issue was isolated to Tailscale's DNS override; disabling that override allowed queries to reach Pi-hole successfully.

**Key conclusion:** Restoring power did not automatically restore the complete service chain. The exercise demonstrated troubleshooting and manual recovery, **not** verified unattended failover or complete disaster recovery.

## Environment and Dependency Map

| Component | Purpose | Recovery relevance |
|---|---|---|
| Nodes A–D | Proxmox compute and cluster membership | Host and quorum state affect management and guest operations |
| OPNsense VMs | Virtual firewall and routing services | Reachability depended on guest startup and network paths |
| Pi-hole containers | DNS filtering and resolution | Client resolution depended on a reachable resolver |
| VLAN-aware switching / Linux bridges | Network transport and segmentation | Link and tagging errors can prevent management or guest traffic |
| Tailscale | Remote access and optional DNS configuration | Client DNS override can change which resolver receives queries |

Internal hostnames, addresses, and unique device identifiers are intentionally omitted.

## Chronological Incident Timeline

The sequence below reflects documented events. **Exact clock times were not recorded**, so no timestamps or downtime duration are inferred.

| Phase | Observed event | Investigation or response |
|---|---|---|
| 1 — Test initiation | Full power-off simulation conducted | Evaluated host and service recovery after restoration |
| 2 — Initial startup | Nodes B and C returned online | Confirmed surviving management access and began checking remaining nodes |
| 3 — Host/network issue | Node A lacked Ethernet link lights and showed quorum-related errors | Investigated link state, cluster reachability, and membership |
| 4 — Firewall disruption | OPNsense VMs were not reachable | Checked network dependencies; no automatic failover claimed |
| 5 — DNS recovery | Pi-hole service established on Node C using a replacement container | Restored a reachable DNS path and configured client resolver options |
| 6 — Client verification | Desktop DNS behavior remained inconsistent | Compared name resolution, network routes, and Tailscale settings |
| 7 — Configuration correction | Tailscale DNS override was identified as a desktop conflict | Disabled the override and confirmed Pi-hole queries succeeded |
| 8 — Post-test status | DNS resolution and routing were restored | Deferred broader unattended recovery and restore validation |

## Technical Investigation

### Symptom 1: Node A did not return with expected network connectivity

**Evidence:** Missing physical link lights and reported quorum errors after power restoration.

**Interpretation:** The symptoms establish a connectivity/cluster-membership problem, but do **not** prove a failed NIC, switch port, cable, or specific Corosync defect. Physical link failure and quorum behavior are separate observations that need independent testing.

**Response:** Compared reachable nodes, checked network availability, and investigated cluster state. A definitive hardware-level cause was not established in the available record.

### Symptom 2: Virtualized firewall services were unreachable

**Evidence:** OPNsense VMs were inaccessible during the recovery period.

**Interpretation:** Virtualized routing introduces a startup and dependency chain involving host availability, guest state, bridge configuration, and physical switching. The exact failed link in that chain was not conclusively isolated.

**Response:** Prioritized restoration of usable network and DNS paths before treating the cluster as recovered.

### Symptom 3: DNS service required restoration

**Evidence:** A replacement Pi-hole container was deployed on Node C, with resolver behavior subsequently tested.

**Response:** Brought up Pi-hole, configured upstream resolvers and client DNS choices, and verified name resolution and routing. The exercise demonstrated a workable manual recovery path, not guaranteed automatic DNS failover for every client.

### Symptom 4: Desktop DNS did not behave as expected

**Evidence:** Desktop queries did not consistently use the intended Pi-hole resolver while Tailscale DNS override was enabled. Disabling the override restored successful Pi-hole queries.

**Supported cause:** A **client-side Tailscale DNS configuration conflict** contributed to the desktop's resolver issue.

**Important distinction:** This finding explains the desktop DNS symptom; it does **not** explain Node A's missing link lights or all OPNsense reachability problems.

## Root-Cause Assessment

| Finding | Confidence | Basis |
|---|---|---|
| Desktop DNS conflict involved Tailscale DNS override | Confirmed for that client symptom | Behavior changed after disabling the override and successful Pi-hole queries followed |
| Node A had a physical-link or interface-connectivity problem | Observed symptom; underlying cause undetermined | No link lights after restoration |
| Cluster quorum was affected during recovery | Observed symptom | Quorum-related errors reported |
| OPNsense reachability depended on unresolved startup/network conditions | Plausible dependency; exact cause unconfirmed | Virtual firewall services were unreachable |
| The entire outage had one common root cause | **Not established** | Evidence supports multiple interacting issues |

## Corrective Actions Taken

- Restored a functional DNS service on Node C.
- Verified DNS queries and routing after recovery changes.
- Disabled Tailscale's desktop DNS override to eliminate the observed resolver conflict.
- Documented the separation between cluster quorum, physical connectivity, firewall reachability, and client DNS.
- Retained the incident as evidence for improving recovery procedures rather than claiming a passed HA test.

## Preventive and Follow-Up Actions

These are **recommendations**, not completed tasks:

1. **Document boot dependencies:** Record which hosts, network paths, firewall guests, and DNS containers must start before normal access returns.
2. **Verify Node A link recovery:** Check cabling, switch-port link, interface state, firmware/BIOS power settings, and system logs during a controlled maintenance window.
3. **Capture quorum evidence:** Save sanitized `pvecm status`, Corosync logs, and host network-state outputs during the next test.
4. **Standardize client DNS behavior:** Document whether Tailscale DNS override should be enabled per device and test Pi-hole selection.
5. **Test service-specific recovery:** Measure time to restore routing, DNS, management access, and representative applications separately.
6. **Validate backups:** Inventory guest coverage and perform an isolated restore test; successful scheduled jobs alone are insufficient.
7. **Repeat the power-loss exercise only with a written rollback plan:** Define expected startup order, checkpoints, and acceptance criteria.

## Validation Matrix

| Capability | Status based on this incident |
|---|---|
| Nodes B and C returned after power restoration | Observed |
| Node A recovered without intervention | Not demonstrated |
| Cluster recovered with no quorum disruption | Not demonstrated |
| OPNsense virtual firewall automatically failed over | Not demonstrated |
| Pi-hole DNS restored through manual recovery | Observed |
| Desktop DNS conflict resolved by changing Tailscale setting | Verified for tested client |
| All clients automatically switched DNS resolvers | Not verified |
| Full application recovery time measured | Not recorded |
| Backup restore and disaster-recovery readiness | Not tested in this exercise |

## Lessons Learned

**Power restoration is not service restoration.** Virtualized network services create dependencies that must be assessed from physical links upward.

**Treat symptoms independently.** A DNS override on one client can coexist with host-link and quorum issues; correcting one does not establish a root cause for the others.

**Define recovery success by service.** A reachable Proxmox host does not prove routing, DNS, guest workloads, or remote access have recovered.

**Report evidence boundaries.** This exercise supports practical incident response and troubleshooting experience, not claims of seamless high availability.

## Related Work

- [Four-Node Proxmox Operations and Recovery](four-node-cluster-operations.md)
- [Proxmox Project Overview](../README.md)
- [Pi-hole DNS](../../../network/pihole-dns/)
- [OPNsense and VLAN Architecture](../../../network/opnsense-vlan-architecture/)
- [Monitoring and Dashboards](../../../monitoring/prometheus-grafana/)

*Public documentation uses generic node aliases and excludes private hostnames, full internal IPs, credentials, MAC addresses, and other unique identifiers.*
