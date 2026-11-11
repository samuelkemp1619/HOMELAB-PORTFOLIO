# Windows Server 2025 Active Directory Lab

**Status:** Active Directory deployment and domain-client authentication validated; Group Policy configuration paused (November 11, 2026).

## Objective

Deploy a Windows Server 2025 Active Directory Domain Services (AD DS) environment on the existing four-node Proxmox VE cluster, integrate a Windows 11 Pro test client, and validate centralized authentication using a standard (non-administrator) domain account.

## Environment

- **Hypervisor:** Proxmox VE, Windows Server and Windows 11 virtual machines hosted on the PowerEdge virtualization node.
- **Domain controller:** Windows Server 2025 Standard, running AD DS and Windows DNS.
- **Test workstation:** Windows 11 Pro with VirtIO network and guest tools.
- **Directory namespace:** Internal, non-public lab DNS domain (intentionally omitted).
- **Network:** Existing management network with OPNsense as gateway and Dnsmasq DHCP. Domain client configured to query the domain controller's DNS service.

Internal IP addresses, VM names, usernames, and domain identifiers are deliberately excluded from this public case study.

## Implementation

1. Installed and configured AD DS, promoted the Windows Server VM as the first domain controller in a new forest, and enabled Windows DNS.
2. Configured AD-integrated DNS zones, secure dynamic updates, and forwarders to existing internal DNS resolvers.
3. Investigated an initial domain-controller connectivity diagnostic failure caused by a missing DNS host (A) record. Re-registered DNS and validated domain-controller discovery.
4. Installed Windows 11 Pro and VirtIO tools in a separate VM. Corrected a client connectivity problem caused by a VLAN tag on a VM attached to an untagged, non-VLAN-aware host bridge.
5. Pointed the Windows 11 DNS client at the domain controller, verified domain-controller discovery, joined the workstation to the domain, and rebooted.
6. Created a dedicated users Organizational Unit (OU) and a standard domain test account.
7. Successfully signed in to the domain-joined Windows 11 workstation through the Proxmox console using the standard account.

## Validation and results

| Test | Observed result |
|---|---|
| Domain-controller discovery | Passed using Windows domain-controller lookup |
| Domain-controller connectivity diagnostic | Passed after DNS host-record registration |
| Basic Windows DNS diagnostic | Overall pass; an AD-zone warning was noted for later review |
| AD DNS zone configuration | AD-integrated forward zones present with secure updates |
| SYSVOL and NETLOGON shares | Confirmed available |
| Windows 11 domain join | Successful |
| Domain administrator authentication | Successful on domain-joined test workstation |
| Standard domain user console sign-in | Successful |
| User identity and local privilege context | Domain identity verified; local Users group and medium integrity level observed; local Administrators group not listed |
| Standard user RDP sign-in | Not authorized by default; console sign-in used instead |
| Group Policy Object deployment | Not started; paused before opening Group Policy Management |

The group-token output did not explicitly show Domain Users membership, so this was not recorded as separately verified. Initial directory service diagnostic warnings were not treated as fully resolved solely because subsequent authentication succeeded.

## Security and operational notes

- Standard domain accounts were tested without granting local administrator rights.
- Remote Desktop access was not broadened solely to make the test pass.
- Public documentation does not include credentials, internal DNS names, precise host IP addresses, or machine identifiers.
- Existing network-wide DHCP/DNS settings were not changed to accommodate the test client.
- A backup of the Windows Server VM was created and archive presence verified; restore testing has not been completed.

## Next steps — paused

- Open Group Policy Management on the domain controller.
- Create a narrowly scoped test GPO linked to the lab users OU.
- Validate GPO application on the Windows 11 domain client.
- Revisit outstanding DNS diagnostic warnings as appropriate.

**Checkpoint:** Pause at Group Policy Management setup, before creating or linking any GPO.
