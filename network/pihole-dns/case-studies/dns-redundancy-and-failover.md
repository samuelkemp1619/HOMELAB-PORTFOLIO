# Case Study: Redundant Pi-hole DNS, Configuration Synchronization, and Failover Testing

**Focus:** DNS availability, network services, Linux administration, validation, and security controls  
**Environment:** Proxmox VE, Pi-hole v6.7.1, OPNsense DHCP, Nebula Sync v0.11.3  
**Validation date:** October 30, 2026  
**Status:** Dual resolvers operational; selected Linux fallback passed; Windows automatic fallback and complete network-wide DNS enforcement not verified

## Problem and Objectives

A homelab relying on a single DNS filtering service can lose name resolution when the host, container, or network path becomes unavailable. The objective was to deploy two independent Pi-hole resolvers on separate virtualization hosts, keep their supported configurations synchronized, advertise both through DHCP, and test how clients behave when a preferred resolver fails.

This design addresses **resolver redundancy**, but two advertised DNS addresses do not guarantee every operating system will fail over promptly.

## Architecture

| Component | Public location | Function |
|---|---|---|
| Primary Pi-hole | Node A, LXC 101 | DNS filtering and authoritative configuration source for synchronization |
| Secondary Pi-hole | Node C, LXC 105 | Independently reachable DNS filtering service |
| Nebula Sync | Node B, monitoring/application container | Hourly primary-to-secondary configuration synchronization |
| OPNsense DHCP | Segmented client networks | Advertises both resolver addresses via DHCP option 6 |
| Cloudflare upstreams | External public DNS | Configured upstream recursion for both Pi-hole instances |

The four configured client networks include management, trusted, IoT, and guest segments. Public documentation masks internal host addresses and excludes credentials.

### Design rationale

- **Separate Proxmox hosts:** A single host outage should not automatically remove both DNS instances.
- **Independent Pi-hole containers:** DNS queries can be answered by either service without requiring the synchronization service to be online.
- **Nebula Sync:** Reduce configuration drift for supported Pi-hole settings and gravity data.
- **Dual DHCP resolvers:** Give clients two configured DNS destinations while recognizing that resolver-selection and retry policies vary by platform.
- **Monitoring and backups:** Check service health and retain a separate secondary configuration export before potentially destructive full synchronization.

## Implementation

Both Pi-hole instances were configured with upstream DNS servers `1.1.1.1` and `1.0.0.1`. OPNsense DHCP option 6 was configured to advertise both Pi-hole addresses on the documented networks. All four Proxmox nodes were configured to list both DNS resolvers.

Nebula Sync v0.11.3 was deployed in a Docker container on Node B. The configuration uses a protected local environment file, full primary-to-replica synchronization, gravity processing, an hourly schedule, and a restart policy. The initial synchronization reported successful authentication, configuration transfer, gravity processing, and completion. The container subsequently reported healthy.

**Change-management precaution:** A Teleporter export of the secondary was downloaded before the first full synchronization because full synchronization can overwrite the secondary's configuration.

## Validation Evidence

| Test | Observed result | Interpretation |
|---|---|---|
| Direct query to primary for a normal domain | Valid DNS response | Primary independently resolves |
| Direct query to secondary for a normal domain | Valid DNS response | Secondary independently resolves |
| Direct query to each instance for an advertising domain | Configured blocking responses | Filtering worked on both |
| Compare gravity domain datasets | Matching sorted-domain SHA256 and 2,135,103 entries on each | Data consistency confirmed at time of comparison |
| Compare adlists | Six matching configured adlists | Supported list configuration aligned |
| Nebula Sync initial run | Completed successfully and container healthy | Initial synchronization operational |
| Windows DHCP inspection | Both resolver addresses present | DHCP advertised redundancy, not failover behavior |
| Linux resolver with unreachable first DNS server | Name resolution succeeded via reachable second server | Controlled Linux fallback passed |
| Windows primary-outage test | Default `nslookup` timed out | Windows automatic fallback **not demonstrated** |

### Controlled Linux fallback — passed

A Linux resolver test used documentation-only address `192.0.2.1` as an intentionally unreachable first server and the reachable secondary Pi-hole as the second server. With `timeout:1 attempts:2`, `getent ahostsv4 example.com` returned valid IPv4 records.

This confirms fallback **under that Linux resolver configuration and test condition**. It does not establish automatic failover on Windows, all DHCP clients, or every real outage scenario.

### Test complication and recovery

The resolver test initially used a mount namespace that isolated the mount but not the underlying `/etc/resolv.conf` file contents. This unintentionally modified Node A's resolver configuration. The original two-resolver configuration was restored, and follow-up direct Pi-hole queries and system name resolution succeeded.

**Lesson:** A mount namespace alone does not guarantee file-content isolation when the same underlying inode is modified. Future resolver experiments should use a genuinely separate file/inode or disposable test environment.

### Windows fallback — not passed

During an earlier controlled primary-Pi-hole stop test, Windows `nslookup` without an explicit server timed out. The primary was restarted and name resolution recovered. Therefore, Windows automatic failover remains **unverified**, despite both DNS addresses being advertised by DHCP and the later Linux fallback success.

## DNS Bypass Controls: Configured, Partially Validated

OPNsense rules were added to allow approved Pi-hole destinations on TCP/UDP 53 and block other direct DNS traffic on port 53 and DNS-over-TLS on TCP 853 across the documented client networks.

From one LAN Windows client, direct Pi-hole queries succeeded, an external direct-DNS query timed out, and a TCP 853 connectivity test failed. These observations are consistent with the intended controls, but the port 853 result alone does not prove which device blocked the connection.

**Validation gaps:** Trusted, IoT, and guest client-path tests remain pending. DNS-over-HTTPS on port 443 is not comprehensively blocked by these rules. IPv6 coverage is not fully audited. The bypass-enforcement project remains paused.

## Risk Register and Follow-Up

| Risk or gap | Impact | Recommended next step |
|---|---|---|
| Windows does not demonstrably fall back | Some clients may lose DNS during primary outage | Re-test with controlled packet capture and resolver timeout observations |
| Secondary not in reviewed scheduled backup jobs | Recovery gap despite DNS redundancy | Add or verify secondary LXC backup coverage |
| Full sync can overwrite secondary configuration | Propagation of bad configuration | Preserve Teleporter exports and test rollback |
| DNS bypass rules not fully tested on all VLANs | Incomplete enforcement evidence | Validate from representative clients on each segment |
| DoH and IPv6 not fully addressed | Potential policy bypass | Define explicit coverage and test plan |
| Synchronization drift over time | Different filtering behavior | Monitor scheduled runs and periodically compare datasets |

## Outcome and Skills Demonstrated

The deployment established two independently functioning DNS filtering services on separate Proxmox hosts, configured hourly synchronization, and confirmed matching filtering data. A controlled Linux fallback scenario passed. A Windows primary-outage test did **not** demonstrate automatic fallback, making client-specific validation a necessary next step.

This work demonstrates DNS administration, OPNsense DHCP integration, Proxmox LXC operations, Docker service deployment, configuration synchronization, careful testing, troubleshooting, and accurate reporting of limitations.

## Related Documentation

- [Pi-hole DNS project overview](../README.md)
- [OPNsense VLAN architecture](../../opnsense-vlan-architecture/)
- [Proxmox cluster operations](../../../virtualization/proxmox/case-studies/four-node-cluster-operations.md)
- [Power-loss recovery incident](../../../virtualization/proxmox/case-studies/power-loss-recovery-incident.md)
- [Monitoring project](../../../monitoring/prometheus-grafana/)

*Public-facing hostnames, internal IP address suffixes, device identifiers, and credentials are omitted or masked.*
