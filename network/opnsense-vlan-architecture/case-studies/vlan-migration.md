# Case Study: OPNsense VLAN Network Migration

## Executive Summary

This case study documents the migration of my homelab from a largely flat network into a segmented architecture using OPNsense, IEEE 802.1Q VLANs, a managed Ethernet switch, and VLAN-aware Proxmox networking.

The original objective was to separate infrastructure management from trusted client traffic and establish a scalable foundation for additional network segmentation. Subsequent deployments expanded that architecture to IoT/guest networks and a dedicated unrestricted-internet wireless VLAN.

The project required coordinating configuration across multiple layers:

- OPNsense
- Proxmox VE
- Linux bridges
- Managed switching
- VLAN tagging
- DHCP
- Firewall rules
- Physical switch ports
- Cluster networking

The migration was completed incrementally to minimize the risk of losing access to the Proxmox cluster.

---

# Environment

The current environment consists of a four-node Proxmox VE cluster connected through managed Ethernet switching.

| Component | Role |
|---|---|
| OPNsense | Router, firewall, VLAN gateway, and DHCP services |
| Proxmox VE | Virtualization and cluster platform |
| Managed Switch | Layer 2 VLAN switching |
| Pi-hole | Redundant DNS filtering |
| Node A | Dell OptiPlex 3000 |
| Node B | HP Elite Mini 600 G9 |
| Node C | HP Elite Mini 600 G9 |
| Node D | Dell PowerEdge R640 |

The original migration was performed during an earlier stage of the homelab's development. The environment subsequently expanded to four Proxmox nodes.

Public node aliases are used instead of internal hostnames.
---

# Original Network

Before the migration, infrastructure management primarily used the existing untagged LAN.

The Proxmox bridge was configured with a static address on the original network and used the existing router as its default gateway.

While functional, this design provided limited segmentation between:

- Hypervisor management
- Infrastructure services
- Trusted clients
- IoT and other client-device categories
- Security-testing systems
- Self-hosted services

A more structured network design was needed as the homelab expanded.

---

# Design Goals

The migration had several objectives.

## Segmentation

Separate infrastructure management from normal client traffic.

## Centralized Routing

Use OPNsense as the primary Layer 3 gateway between network segments.

## Firewall Control

Route inter-VLAN traffic through OPNsense so communication between networks can be explicitly controlled.

## Expandability

Provide a foundation for additional networks without redesigning the entire environment.

## Maintainability

Use consistent VLAN assignments and documented switch-port behavior.

## Availability

Perform the migration incrementally to reduce the possibility of losing administrative access to all Proxmox nodes simultaneously.

---

# VLAN Architecture

The original migration established two primary network segments:

| VLAN | Purpose |
|---|---|
| VLAN 10 | Infrastructure management |
| VLAN 20 | Trusted client devices |

The architecture was subsequently expanded to support additional segmented networks.

The current configuration includes VLANs 10, 20, 30, 40, 50, and 100.

These VLANs support management, trusted devices, IoT/guest segmentation, an unrestricted-internet wireless network, and additional infrastructure connectivity.

VLAN 50 provides internet access without Pi-hole filtering and restricts access to internal private networks.

The initial VLAN migration and the later VLAN 50 deployment are documented as separate implementation milestones.
---

# OPNsense Deployment

OPNsense runs as a virtual machine inside the Proxmox environment.

Its responsibilities include:

- Routing
- Firewall policy
- VLAN gateways
- DHCP
- Inter-VLAN routing
- Network segmentation

Virtualizing the firewall allows the networking environment to remain integrated with the Proxmox infrastructure while still providing centralized routing and policy enforcement.

---

# VLAN-Aware Proxmox Networking

The Proxmox Linux bridge was configured to support VLAN-aware networking.

Conceptually:

```text
Physical NIC
     |
     v
VLAN-Aware Linux Bridge
     |
     +---- Proxmox Management
     |
     +---- OPNsense
     |
     +---- Virtual Machines
     |
     +---- LXC Containers
```

The bridge allows tagged VLAN traffic to pass between the physical network and virtualized workloads.

The VLAN-aware configuration supports a broad VLAN range so additional networks can be introduced later without redesigning the bridge.

---

# Managed Switch

The Proxmox nodes connect through a TP-Link managed Ethernet switch.

The switch provides the Layer 2 VLAN functionality required to carry tagged and untagged traffic between:

- Proxmox hosts
- OPNsense
- Administrative devices
- Other network equipment

Switch configuration must match the tagging expectations of both Proxmox and OPNsense.

A VLAN can be correctly configured on the firewall and hypervisor while still failing if the physical switch port is configured incorrectly.

---

# Tagged vs. Untagged Traffic

Understanding tagged and untagged Ethernet traffic was an important part of the migration.

## Tagged Traffic

Tagged traffic includes an IEEE 802.1Q VLAN identifier.

Tagged links can transport multiple VLANs over the same physical Ethernet connection.

These links are useful between infrastructure devices such as:

```text
OPNsense
    |
 VLAN trunk
    |
Managed Switch
    |
 VLAN trunk
    |
Proxmox
```

## Untagged Traffic

Untagged traffic does not contain an 802.1Q VLAN tag.

Access ports typically place untagged client traffic into a specific VLAN using the port's PVID configuration.

This allows devices that are unaware of VLANs to participate in a segmented network.

---

# PVID

The Port VLAN ID determines which VLAN receives incoming untagged traffic on a switch port.

Correct PVID configuration was important during the migration because an incorrect PVID could place a device into the wrong network or prevent it from reaching its gateway.

The switch configuration therefore had to account for both:

- VLAN membership
- PVID behavior

---

# Management VLAN Migration

The Proxmox management interfaces were migrated to VLAN 10.

The resulting management design became:

```text
                 Management VLAN
                       |
        --------------------------------
        |              |               |
        v              v               v
    Node A       Node B       Node C
```

OPNsense provides the gateway for this network.

Moving management traffic into its own VLAN separates hypervisor administration from ordinary trusted client traffic.

---

# Proxmox Cluster Considerations

Migrating the Proxmox nodes required additional care because the systems participate in a Corosync cluster.

Changing management addresses without updating cluster communication could result in:

- Lost node communication
- Corosync failures
- Loss of quorum
- Management-interface loss

The migration therefore required validating cluster state throughout the process.

Useful validation included:

```bash
pvecm status
```

and reviewing the Corosync configuration.

---

# Corosync Migration

Cluster communication was updated to use the new management network.

The Corosync ring addresses were migrated so each node communicated using its management VLAN address.

The configuration version was incremented as the cluster configuration changed.

After migration, cluster membership was verified to ensure:

- All expected nodes were present
- All expected votes were available
- Quorum was maintained
- Node names resolved to the expected cluster members

---

# DHCP

OPNsense provides DHCP services using **DNSMasq**.

DHCP configuration is associated with the applicable network interfaces and VLAN segments.

Validation includes confirming that clients receive:

- An address from the correct network
- The expected subnet mask
- The appropriate default gateway
- The configured DNS resolvers

Infrastructure systems requiring predictable addressing use controlled addressing configurations.

During the later VLAN 50 deployment, DNSMasq interface selection was identified as a cause of DHCP failure. Correcting the configuration restored successful client address assignment.

That troubleshooting incident is documented separately in the unrestricted wireless VLAN 50 case study.
---

# Trusted VLAN

VLAN 20 was introduced for trusted client devices.

This created a logical separation between:

```text
Infrastructure Management
          |
      Firewall
          |
    Trusted Clients
```

Trusted devices therefore do not need to share the same Layer 2 management network as the Proxmox hypervisors.

---

# Firewall Architecture

OPNsense acts as the policy enforcement point between VLANs.

The deployed architecture uses OPNsense to enforce firewall policies between VLANs:

```text
Source Network
      |
      v
OPNsense Firewall
      |
   Policy Check
      |
      v
Destination Network
```

Traffic between VLANs can therefore be explicitly permitted or denied according to purpose.

This provides a stronger security model than allowing unrestricted communication between all devices.

---

# DNS Integration

Two Pi-hole instances provide DNS filtering on separate Proxmox nodes.

- **Node A:** Primary Pi-hole
- **Node C:** Secondary Pi-hole

OPNsense DHCP configuration advertises the appropriate DNS resolver addresses to applicable client networks.

Independent DNS resolution and selected resolver-fallback behavior have been verified.

The dedicated unrestricted wireless VLAN 50 uses external DNS resolvers rather than Pi-hole.

DNS availability remains an important operational consideration because a DNS failure can resemble a broader network connectivity failure.

Automatic resolver failover has not been verified across every client platform.
---

# Migration Strategy

The network was migrated incrementally rather than changing every system simultaneously.

The general process was:

1. Configure OPNsense.
2. Create the management VLAN.
3. Configure VLAN-aware Proxmox networking.
4. Configure switch VLAN membership.
5. Configure port PVID behavior.
6. Migrate infrastructure systems.
7. Validate connectivity.
8. Update cluster communication.
9. Verify Corosync.
10. Verify quorum.
11. Introduce the trusted VLAN.
12. Test client connectivity.
13. Validate DNS.
14. Validate routing and firewall behavior.

This approach reduced the chance that a single configuration mistake would make the entire environment inaccessible.

---

# Troubleshooting

The migration required troubleshooting across several network layers.

When connectivity failed, the problem could potentially exist at:

```text
Application
    |
Operating System
    |
IP Configuration
    |
Proxmox Bridge
    |
802.1Q Tagging
    |
Physical NIC
    |
Switch Port
    |
VLAN Membership
    |
PVID
    |
OPNsense Interface
    |
Firewall Policy
    |
Routing
```

Troubleshooting therefore required validating each layer rather than assuming OPNsense was responsible for every connectivity problem.

---

# Validation

After configuration changes, connectivity was tested between relevant systems.

Validation included:

- Proxmox management access
- Gateway connectivity
- Inter-node connectivity
- Corosync membership
- Cluster quorum
- DHCP
- DNS
- Trusted-network connectivity
- Switch-port behavior

Cluster health was checked using:

```bash
pvecm status
```

Successful connectivity between nodes and their gateway confirmed that the management network was operational.

---

# Result

The migration successfully established:

- Dedicated management VLAN
- Separate trusted-client VLAN
- OPNsense inter-VLAN routing
- Centralized firewall policy
- VLAN-aware Proxmox networking
- Managed-switch VLAN configuration
- Functional Proxmox cluster communication
- Working Corosync communication
- Pi-hole DNS integration
- Foundation that supported subsequent deployments of additional segmented networks, including VLAN 50

The Proxmox nodes now operate on the dedicated management network while trusted client devices can operate on a separate VLAN.

---

# Security Improvements

The segmented architecture provides several security benefits.

## Reduced Management Exposure

Hypervisor management interfaces no longer need to share the same Layer 2 network with all normal client devices.

## Firewall Enforcement

Traffic crossing VLAN boundaries can be inspected and controlled by OPNsense.

## Improved Device Classification

Systems can be grouped according to their role and trust level.

## Additional Network Isolation

The VLAN architecture has been extended beyond the original management and trusted networks.

Additional segmentation supports IoT and guest networks, while VLAN 50 provides unrestricted internet access without permitting normal access to internal private networks.

Firewall rules on OPNsense control communication across these network boundaries.

The documented VLAN 50 deployment includes successful internet connectivity and internal-network isolation testing.

## Better Monitoring

Traffic boundaries make unusual communication patterns easier to identify and investigate.

---

# Availability Considerations

OPNsense currently provides critical routing services for the environment.

Because the active OPNsense instance is virtualized, hypervisor maintenance must consider the effect on:

- Internet connectivity
- Inter-VLAN routing
- DHCP
- Firewall services
- Remote administration

Standby OPNsense VMs exist on other cluster nodes, but automatic failover should not be assumed until it has been explicitly configured and tested.

---

# Remote Administration

Tailscale complements the VLAN architecture by providing secure remote access to authorized internal networks.

The combined architecture is:

```text
Remote Administrator
        |
        v
Tailscale
        |
        v
Subnet Router
        |
        v
Management VLAN
        |
        v
Proxmox / Infrastructure
```

This allows remote management without exposing Proxmox administrative interfaces directly to the public Internet.

---


---

# Lessons Learned

## VLAN Configuration Spans Multiple Systems

Creating a VLAN on OPNsense alone is not enough.

The VLAN must be correctly represented across:

- Firewall
- Hypervisor
- Linux bridge
- Virtual interfaces
- Physical switch
- Switch ports
- Client configuration

## PVID Matters

A port can have correct VLAN membership but still behave incorrectly if its PVID does not match the intended untagged network.

## Cluster Networking Requires Additional Planning

Changing Proxmox management networking also affects Corosync and cluster communication.

Cluster health must therefore be considered during network migrations.

## DNS Can Look Like a Network Failure

A client may have functional Layer 3 connectivity while applications fail because DNS resolution is unavailable.

## Incremental Changes Reduce Risk

Migrating one portion of the environment at a time made troubleshooting easier and reduced the risk of losing access to the entire cluster.

## Documentation Is Part of Network Administration

Maintaining a record of:

- VLAN IDs
- Network purposes
- Switch ports
- Gateways
- Firewall policy
- Infrastructure dependencies

makes future troubleshooting and expansion significantly easier.

---

# Skills Demonstrated

This project demonstrates practical experience with:

- OPNsense
- Proxmox VE networking
- VLANs
- IEEE 802.1Q
- Managed Ethernet switching
- Tagged and untagged Ethernet
- PVID configuration
- Linux bridges
- Inter-VLAN routing
- Firewall policy
- DHCP
- DNS
- Pi-hole
- Corosync
- Proxmox cluster networking
- Network segmentation
- Network troubleshooting
- Secure remote administration
- Change management
- Infrastructure documentation

---

# Project Status

**Completed — VLAN Migration and Segmented Network Deployment**

The original migration successfully established dedicated management and trusted-client VLANs, OPNsense routing and firewall enforcement, VLAN-aware Proxmox networking, and functional Corosync cluster communication.

The homelab subsequently expanded to a four-node Proxmox VE cluster with additional VLAN segmentation, including the dedicated unrestricted-internet wireless VLAN 50.

The deployed architecture includes DNSMasq DHCP services, redundant Pi-hole DNS filtering, and Tailscale remote administration.

**Validation limitations:**

- Automatic OPNsense firewall failover has not been configured and verified as a completed high-availability capability.
- DNS resolver failover has not been validated across every client platform.
- Comprehensive validation of every inter-VLAN firewall rule and client-isolation scenario is not claimed.

The case study documents completed migration work and subsequent deployed network improvements, not unfinished infrastructure plans.
