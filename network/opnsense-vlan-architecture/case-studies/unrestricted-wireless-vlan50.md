# Unrestricted Internet Wireless VLAN — Deployment and DHCP Troubleshooting

**Completed:** October 31, 2026  
**Status:** Operational; deployment validation confirmed complete by the operator on October 31, 2026. Packet captures and client diagnostic evidence are documented below; additional detailed test logs were not supplied.

## Objective

Provide a dedicated wireless SSID for selected devices requiring ordinary, unfiltered internet access without the existing Pi-hole DNS policy. Keep these clients separated from the homelab's internal services and management networks.

“Unrestricted” refers to internet/DNS policy, **not** permission to access internal infrastructure.

## Components and topology

- **Wireless:** Grandstream GWN7660E; dedicated SSID `Skemp_UNSECURE`, bridged client IP assignment, IEEE 802.1Q VLAN 50, 2.4 GHz and 5 GHz.
- **Switching:** TP-Link TL-SG108E v6.0; AP connected at switch port 2, with VLAN 50 tagged on port 2 and the tagged Proxmox-facing uplinks.
- **Hypervisor:** Proxmox VE, Node A; VLAN-aware Linux bridge `vmbr0` with physical `nic0`.
- **Firewall/router:** OPNsense VM 100; added sixth VirtIO adapter `net5`, Proxmox `tag=50`, appearing in OPNsense as `vtnet5` / interface `UNSECURE`.
- **Addressing:** IPv4 subnet `192.168.50.0/24` (public documentation of network prefix only), OPNsense gateway `192.168.50.1`, DHCP pool `192.168.50.100–200`.
- **DHCP:** OPNsense Dnsmasq; `UNSECURE` explicitly selected in its interface list.
- **Client DNS policy:** DHCP option 6 advertises Cloudflare `1.1.1.1` and `1.0.0.1`, rather than Pi-hole.

No SSID passphrase, device MAC addresses, or individual client IP addresses are recorded here.

## IPv4 firewall policy

The UNSECURE interface rules were configured in this order:

1. **Pass** client DHCP requests (UDP source port 68 to destination port 67).
2. **Block** access from UNSECURE network to **This Firewall**.
3. **Block** access from UNSECURE network to the `PRIVATE_NETWORKS` alias.
4. **Pass** remaining IPv4 traffic from UNSECURE network to any destination.

OPNsense automatic outbound Source NAT includes the UNSECURE network. Existing LAN/TRUSTED/IoT/GUEST DNS enforcement policies were not extended to this SSID.

**Caveats:** Rule ordering and alias membership matter. The scope of `PRIVATE_NETWORKS` must be reviewed if internal address ranges change. These are IPv4 rules; IPv6 firewalling/routing was not fully validated. A Cloudflare DoH result from one client does not prove every client or application bypasses Pi-hole.

## Troubleshooting timeline

1. The test phone associated to the SSID but self-assigned an APIPA `169.254.x.x` address, indicating that it had not received a DHCP lease.
2. An initial OPNsense packet capture did not show DHCP requests on UNSECURE.
3. `tcpdump` on Proxmox physical `nic0` showed tagged VLAN 50 DHCP Discover broadcasts.
4. `tcpdump` on VLAN-aware `vmbr0` showed the same type of tagged DHCP traffic.
5. QEMU `info network` confirmed VM 100's sixth VirtIO adapter and `tap100i5`.
6. `bridge vlan show` confirmed `tap100i5` was VLAN 50 PVID/egress-untagged; `nic0` permitted VLANs 2–4094. Bridge forwarding and VLAN 50 FDB learning were confirmed.
7. A capture directly on `tap100i5` **without a VLAN filter** revealed untagged DHCP requests from the test phone. No DHCP offers were observed during the capture.
8. OPNsense Dnsmasq General settings revealed the root cause: `UNSECURE` had been omitted from its selected interfaces, even though a DHCP range had already been configured.
9. Adding `UNSECURE` to Dnsmasq's interface selection and applying the change restored client connectivity.

**Root cause:** DHCP service interface selection, **not** missing VLAN tagging on the switch or Proxmox bridge.

## Verification evidence

| Test | Result |
|---|---|
| Client Wi-Fi association | Passed |
| DHCP lease / internet access after Dnsmasq fix | Passed (client reported connected) |
| VLAN 50 tagged traffic on physical uplink and bridge | Passed via packet captures |
| VLAN 50 delivery to VM adapter | Passed via untagged capture on `tap100i5` |
| Internal homelab service access from test phone | Blocked in client test |
| Cloudflare diagnostic: Connected to 1.1.1.1 | Yes |
| Cloudflare diagnostic: DNS over HTTPS | Yes |
| Cloudflare resolver IPv4 reachability | 1.1.1.1 and 1.0.0.1 reachable |
| Cloudflare resolver IPv6 reachability | Not reachable in test |
| DNS policy validation | Operator confirms complete; exhaustive per-application evidence not supplied |
| IPv6 internal-network isolation | Operator confirms deployment validation complete; specific IPv6 test evidence not provided |

## Skills demonstrated

802.1Q trunk/access VLAN design; managed switch configuration; Proxmox VLAN-aware Linux bridges; VirtIO/TAP networking; packet capture at multiple OSI layers; DHCP failure diagnosis; OPNsense interface/firewall/NAT policy; DNS resolver policy separation; evidence-based network validation.

## Validation closeout

The operator confirmed that the deployment's validation was completed after the initial documented tests. Specific additional commands and results were not provided, so this case study records that confirmation without inventing measurements. Revalidate firewall alias coverage if the network design changes.
