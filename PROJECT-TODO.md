# Homelab Master To-Do — November 5, 2026

This document tracks completed homelab work and outstanding projects. Completed items represent implemented work or verified project milestones, not necessarily full validation of every associated capability.

Projects that have been discontinued or explicitly excluded from the portfolio are not included in the active backlog.

## Completed / Operational

- [x] Four-node Proxmox VE cluster established (public aliases: Node A–D).
- [x] November 2026 Proxmox cluster update campaign completed: all four nodes updated, rebooted, and verified quorate.
- [x] Three-tier 3D-printed micro-PC holder: design, printing, assembly, node installation, and cable management (October 19).
- [x] Rack shelf mounting with compatible 12-24 hardware and washers.
- [x] Graphy Raspberry Pi 4 Grafana kiosk and rotating dashboards.
- [x] Vaultwarden PowerShell password audit and CSV (573 logins assessed; findings identified, not automatically remediated).
- [x] Windows Server 2025 lab VM installed and activated; additional lab configuration remains paused.
- [x] Proxmox scheduled backup jobs configured and successful task logs reviewed. Restore validation remains outstanding.
- [x] Tailscale remote access verified on laptop and phone.
- [x] OPNsense VLAN segmentation deployed, including the dedicated unrestricted-internet wireless VLAN 50.
- [x] Redundant Pi-hole DNS services deployed and independent resolution verified.
- [x] Prometheus and Grafana monitoring deployed.

## Outstanding Infrastructure Improvements

- [ ] Expand network switch capacity while preserving VLAN segmentation.
- [ ] Evaluate the spare desktop for dedicated NAS storage.
- [ ] Implement additional storage for Immich and general homelab use.
- [ ] Review existing backup coverage and backup storage capacity.
- [ ] Perform isolated backup restoration testing and document results.
- [ ] Validate remaining DNS enforcement behavior across trusted, IoT, and guest networks, including DNS-over-HTTPS and IPv6 considerations.

## Paused — Resume Only When Requested

- [ ] Minecraft Java public server: WAN/public IPv4 and public access verification.
- [ ] Modular 3D-printed desktop PC test bench supporting ATX, micro-ATX, Mini-ITX, and GPU mounting.
- [ ] Windows Server / Active Directory lab continuation.
- [ ] Additional cybersecurity monitoring and detection.
- [ ] Further network segmentation and infrastructure enhancements.
- [ ] Spotify Car Thing Stream Deck-style repurposing.

## Documentation and Validation Notes

- The Proxmox cluster update campaign is complete; future package and firmware maintenance is separate ongoing administration.
- Successful backup creation does not establish successful restoration.
- Linux DNS resolver fallback has been tested; automatic Windows-client DNS failover remains unverified.
- Some DNS enforcement and client-isolation scenarios require additional validation.
- The Vaultwarden audit identified findings but did not automatically remediate passwords.
- Projects marked paused are not considered active deployments or completed implementations.

## Exclusions

Discontinued projects and previously removed backlog items are intentionally omitted. Historical technical evidence remains in the appropriate completed project documentation.