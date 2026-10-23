# Pi-hole DNS Filtering and Redundancy

## Overview

This project documents a redundant, two-instance Pi-hole DNS filtering deployment for the Proxmox homelab. The design keeps DNS filtering available on a second Proxmox host and uses Nebula Sync to replicate supported Pi-hole configuration from the primary to the secondary.

**Important:** Both DNS servers have been tested independently for resolution and blocking. Automatic client failover has not yet been proven and should not be described as fully validated high availability.

## Technologies

- Pi-hole v6.7.1
- Proxmox VE and LXC
- OPNsense DHCP / DNS options
- Nebula Sync v0.11.3
- Docker Compose
- DNS, VLANs, and Cloudflare upstream resolvers

## Architecture

| Role | Proxmox host | LXC ID | Address |
|---|---|---:|---|
| Primary Pi-hole | HomeProx | CT 101 | `192.168.10.2` |
| Secondary Pi-hole | HomeProx3 | CT 105 | `192.168.10.3` |
| Nebula Sync | HomeProx2 | CT 103 (separate Docker container) | Synchronizes primary to secondary |

Both Pi-hole instances run independently in LXC containers. Nebula Sync runs as its own Docker container inside CT 103 alongside the existing Dashy, Grafana, Prometheus, PVE Exporter, and Pi-hole Stats containers. It is not installed inside the Dashy container.

## DNS Flow and DHCP

OPNsense advertises both Pi-hole addresses through DHCP DNS option 6 on the configured LAN and VLAN networks:

- LAN / VLAN 10
- TRUSTED / VLAN 20
- IOT / VLAN 30
- GUEST / VLAN 40

The advertised DNS servers are `192.168.10.2` and `192.168.10.3`. Proxmox host resolver settings were also updated to list both addresses.

Clients may select or retry DNS servers according to their own resolver behavior. Providing two DNS addresses does not guarantee immediate or consistent failover for every client or application.

## Pi-hole Configuration and Synchronization

Both Pi-hole instances run version **6.7.1**. Their upstream resolvers are configured as Cloudflare `1.1.1.1` and `1.0.0.1`, with listening mode `ALL` and local domain `pi.hole`.

The gravity database checksums were verified to match before synchronization. Nebula Sync v0.11.3 was then deployed in CT 103 using a protected local `.env` file; credentials are not stored in this repository.

The service is configured for:

- Full primary-to-replica synchronization
- Gravity processing after synchronization
- An hourly schedule at minute 0
- Central time zone (`America/Chicago`)
- Container restart policy `unless-stopped`

The initial run reported successful authentication, Teleporter synchronization, configuration synchronization, gravity processing, session invalidation, and `Sync completed`. The container reported healthy after startup.

Full synchronization can replace the secondary's configuration with the primary's configuration. A Teleporter backup of CT 105 was downloaded before the initial run.

## Validation Performed

- Queried `google.com` directly against both `192.168.10.2` and `192.168.10.3`; both returned valid records.
- Queried `doubleclick.net` directly against both instances; both returned the configured blocking response (`0.0.0.0` for A and `::` for AAAA).
- Confirmed Windows receives both DNS addresses from OPNsense DHCP.
- Confirmed all four Proxmox nodes are configured to use both DNS addresses.
- Confirmed both Pi-hole containers and their `pihole-FTL` services are enabled for automatic startup.
- Confirmed Nebula Sync completed an initial full synchronization and its container reported healthy.

### Failover Limitation

A controlled test temporarily stopped primary CT 101. Windows `nslookup` without an explicit server timed out while the primary address was unavailable. The primary was restarted and DNS resolution recovered. Therefore, automatic failover is **not yet validated**; further testing of Windows DNS Client behavior and/or a different failover design remains outstanding. Both Pi-hole instances are currently running.

## Backup and Recovery

Both Pi-hole LXC containers are managed by Proxmox and included in the existing infrastructure backup approach. A Pi-hole Teleporter export of the secondary was downloaded before the initial full sync.

Before future full synchronization or major configuration changes:
1. Confirm the primary configuration is authoritative and correct.
2. Create a current backup of the secondary.
3. Review Nebula Sync logs after the run.
4. Test DNS resolution and blocking against both addresses.
5. Avoid claiming failover is verified until a controlled client-side test succeeds.

## Security Considerations

- Keep Nebula Sync app passwords in a protected local environment file; never commit credentials, tokens, or secrets.
- Limit administrative access to Pi-hole and Nebula Sync.
- Keep DNS services restricted to authorized networks.
- Preserve independent backups of the secondary before full configuration syncs.
- Treat redundancy and automatic client failover as separate capabilities.

## Skills Demonstrated

- Pi-hole v6 administration
- DNS filtering and troubleshooting
- Proxmox LXC administration
- OPNsense DHCP integration
- VLAN-aware DNS deployment
- Docker Compose service deployment
- Credential handling
- Configuration synchronization
- Backup and recovery planning
- Controlled service validation

## Next Steps

- Complete a controlled and repeatable client failover test without leaving DNS unavailable longer than necessary.
- Evaluate whether client resolver behavior meets availability requirements.
- Consider a dedicated DNS VIP or another design only if needed and compatible with the network architecture.
- Monitor Nebula Sync scheduled runs and verify synchronization remains healthy.
- Document recovery from a failed or unwanted full synchronization.
