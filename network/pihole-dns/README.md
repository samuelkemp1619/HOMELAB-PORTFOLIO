# Pi-hole DNS Filtering and Redundancy

## Overview

This project documents a redundant, two-instance Pi-hole DNS filtering deployment for the Proxmox homelab. The design keeps DNS filtering available on a second Proxmox host and uses Nebula Sync to replicate supported Pi-hole configuration from the primary to the secondary.

**Validation status (October 30, 2026):** Both DNS servers resolve independently, and HomeProx's Linux resolver successfully fell back to the secondary during a simulated unavailable-primary test. This does not prove failover for Windows or every DHCP client, or establish fully validated high availability.

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

### Controlled Linux Resolver Failover — Passed (October 30, 2026)

- Confirmed both `192.168.10.2` (CT101/HomeProx) and `192.168.10.3` (CT105/HomeProx3) returned DNS answers for `google.com`.
- Confirmed `192.0.2.1` was unreachable on port 53 (`dig` timed out), making it a suitable simulated failed primary for this test.
- Configured a temporary Linux resolver test with first server `192.0.2.1`, second server `192.168.10.3`, and `timeout:1 attempts:2`. `getent ahostsv4 example.com` returned valid IPv4 addresses, demonstrating fallback to the reachable secondary under these conditions.
- **Test incident and recovery:** The mount namespace isolated the mount but not the underlying `/etc/resolv.conf` file contents; the test unintentionally replaced HomeProx's resolver configuration. Restored the original `search com`, `nameserver 192.168.10.2`, `nameserver 192.168.10.3` settings and verified the restored file contents. Follow-up direct queries to both Pi-holes and system `getent ahostsv4 google.com` all succeeded. Do not reuse that mount-bind test as a safe isolated method without a separate file/inode.
- Confirmed matching gravity domain data: **2,135,103** entries on each instance, with identical sorted-domain SHA256; each has six adlists with matching address/enabled configurations and no custom domainlist entries. DNS-related Pi-hole TOML settings matched in the compared configuration diff.
- Confirmed all four Proxmox nodes previously listed both DNS servers and OPNsense Dnsmasq DHCP option 6 advertises both on LAN, TRUSTED, IOT, and GUEST.

**Scope:** Successful simulated Linux resolver fallback is verified on HomeProx. Real Windows-client automatic failover remains unverified, as does a physical CT101 outage scenario for all clients. DHCP advertising two servers does not guarantee client-specific failover behavior.

### Earlier Windows Failover Limitation

A controlled test temporarily stopped primary CT 101. Windows `nslookup` without an explicit server timed out while the primary address was unavailable. The primary was restarted and DNS resolution recovered. Therefore, **Windows client failover remains unverified**; the later successful Linux resolver test does not override this earlier Windows result. Both Pi-hole instances are currently running.

## Backup and Recovery

Both Pi-hole LXC containers are managed by Proxmox, but the last reviewed scheduled backup jobs included CT101 and **did not include CT105**. Secondary Pi-hole backup coverage needs to be added or separately verified. A Pi-hole Teleporter export of the secondary was downloaded before the initial full sync.

Before future full synchronization or major configuration changes:
1. Confirm the primary configuration is authoritative and correct.
2. Create a current backup of the secondary.
3. Review Nebula Sync logs after the run.
4. Test DNS resolution and blocking against both addresses.
5. Distinguish the passed HomeProx Linux resolver failover test from unverified Windows and other client failover.

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

- Complete controlled Windows and other client failover tests without disrupting production DNS.
- Evaluate whether client resolver behavior meets availability requirements.
- Consider a dedicated DNS VIP or another design only if needed and compatible with the network architecture.
- Monitor Nebula Sync scheduled runs and verify synchronization remains healthy.
- Add or verify scheduled backup coverage for secondary Pi-hole CT105.
- Document recovery from a failed or unwanted full synchronization.

## DNS Bypass Enforcement (Paused)

OPNsense rules were added on LAN, TRUSTED, IOT, and GUEST to make the Pi-hole instances the permitted conventional DNS resolvers and to block common direct-DNS bypass paths.

### Configured rules

- Each interface has an allow rule for the `PiHole_DNS` alias, covering both `192.168.10.2` and `192.168.10.3` on TCP/UDP port 53.
- Each interface has a block rule for other IPv4 TCP/UDP traffic to destination port 53.
- Each interface has a block rule for IPv4 TCP traffic to destination port 853 (DNS-over-TLS).
- On IOT and GUEST, the Pi-hole allow rule remains above the internal-network block, preserving the required DNS exception while retaining network isolation.
- On LAN, an upstream exception was added for the Pi-hole servers to reach the configured Cloudflare upstreams in the `PiHole_Upstreams` alias on port 53.

### Validation performed

- From a Windows client on LAN, direct queries to both Pi-hole addresses returned valid `google.com` records.
- A direct DNS query to `8.8.8.8` timed out, consistent with the external DNS block.
- A TCP connection test to `1.1.1.1:853` failed, consistent with DNS-over-TLS being unreachable from that client. The result alone does not prove which device blocked the connection.
- OPNsense LAN IPv6 configuration was checked and set to `None`; the tested Windows Ethernet adapter had no non-link-local IPv6 address or IPv6 DNS server configured.

### Remaining validation and limitations

- The firewall rules on TRUSTED, IOT, and GUEST have been configured, but client-side enforcement tests are pending because no test device was connected to those networks during this session.
- DNS-over-HTTPS (DoH) on TCP port 443 has not been addressed. Blocking ports 53 and 853 does not prevent every encrypted-DNS method.
- IPv6 settings on the other client networks and clients have not been fully audited.
- The project was paused after configuring the rules and verifying their order. Do not describe network-wide bypass prevention as fully tested or complete.
