# Secure Remote Homelab Administration with Tailscale

## Project Overview

This project documents the implementation of Tailscale as a secure remote-access solution for my Proxmox homelab.

The primary objective is to securely administer internal infrastructure while away from the local network without exposing management interfaces directly to the public Internet.

Tailscale provides an encrypted WireGuard-based overlay network between authorized devices and the homelab.

The current architecture uses a Proxmox host as a Tailscale subnet router, allowing authorized remote devices to reach systems on the management network even when those systems do not run Tailscale themselves.

---

# Environment

The homelab consists of a three-node Proxmox VE cluster and segmented networking managed by OPNsense.

| System | Role |
|--------|------|
| HomeProx | Proxmox host, primary infrastructure, Tailscale subnet router |
| HomeProx2 | Proxmox application host |
| HomeProx3 | Proxmox monitoring host |
| OPNsense | Router, firewall, and VLAN gateway |
| Pi-hole | Internal DNS filtering |
| Laptop | Remote administration endpoint |
| iPhone | Mobile remote administration endpoint |

The environment separates management infrastructure from other network traffic using VLANs and firewall policies.

---

# Problem

Remote administration creates a security challenge.

Management interfaces such as:

- Proxmox Web UI
- SSH
- Grafana
- Dashy
- Pi-hole administration
- Internal infrastructure services

should not be unnecessarily exposed directly to the public Internet.

Traditional remote-access designs may require:

- Publicly exposed VPN services
- Port forwarding
- Dynamic DNS
- Firewall rules allowing inbound Internet traffic

I wanted a solution that provided remote administrative access while minimizing public exposure.

---

# Solution

Tailscale was selected to provide encrypted remote connectivity.

Tailscale creates a private overlay network using WireGuard.

Authorized devices join the private Tailscale network and communicate through encrypted tunnels.

The basic architecture is:

```text
Remote Laptop / Mobile Device
             |
             |
      Encrypted Tailscale
             |
             |
       HomeProx
    Tailscale Node
             |
       Subnet Router
             |
             |
     Management Network
             |
    ---------------------
    |         |         |
HomeProx  HomeProx2  HomeProx3
    |
 Internal Infrastructure
```

This allows remote administration without publishing internal management interfaces directly to the Internet.

---

# Subnet Router Architecture

Not every device on the management network needs to run Tailscale.

HomeProx acts as a Tailscale subnet router.

The subnet router advertises the management network to authorized Tailscale clients.

General traffic flow:

```text
Remote Administrator
        |
        v
Tailscale Overlay Network
        |
        v
HomeProx Subnet Router
        |
        v
Management VLAN
        |
        +---- Proxmox Hosts
        |
        +---- Monitoring
        |
        +---- DNS
        |
        +---- Other Infrastructure
```

This architecture allows existing infrastructure to remain on the internal network while still being reachable through authenticated Tailscale clients.

---

# IP Forwarding

Because HomeProx forwards traffic between the Tailscale overlay and the internal management network, IP forwarding must be enabled.

The forwarding configuration is persisted using:

```text
/etc/sysctl.d/99-tailscale-forwarding.conf
```

This ensures the required forwarding configuration survives a reboot.

---

# Route Advertisement

The HomeProx Tailscale node advertises the management subnet.

The advertised route must also be approved through Tailscale before clients are permitted to use it.

This provides an additional administrative control over which internal networks become reachable through the overlay network.

---

# Remote Access Flow

A remote administrative session follows this general path:

```text
Laptop / Mobile Device
        |
        | Authenticated Tailscale Connection
        v
Tailscale Network
        |
        | Encrypted WireGuard Tunnel
        v
HomeProx
        |
        | Subnet Routing
        v
Management VLAN
        |
        v
Internal Service
```

The internal service does not need to accept connections directly from the public Internet.

---

# Security Benefits

## Reduced Public Exposure

Internal administrative services do not need public port-forwarding rules solely for remote management.

This reduces exposure of services such as:

- Proxmox
- SSH
- Grafana
- Pi-hole
- Internal web interfaces

---

## Encrypted Connectivity

Traffic between Tailscale nodes is protected using WireGuard-based encryption.

This provides encrypted connectivity when administering the homelab from untrusted networks such as:

- Cellular networks
- Public Wi-Fi
- Remote locations

---

## Device-Based Access

Remote access is limited to devices authorized to participate in the Tailscale network.

This provides an additional security boundary compared with simply exposing a management interface to the Internet.

---

## Network Segmentation Remains Relevant

Tailscale does not replace the internal firewall architecture.

OPNsense and VLAN segmentation continue to control how systems communicate inside the homelab.

Remote access should provide only the network access required for administration.

---

# Administrative Endpoints

The remote-access design is intended to support administration from devices such as:

### Laptop

Primary remote systems-administration workstation.

Potential administrative tasks include:

- Proxmox management
- SSH administration
- Monitoring access
- Infrastructure troubleshooting

### iPhone

Provides mobile access for checking infrastructure while away from the network.

Potential uses include:

- Checking dashboards
- Reviewing service status
- Performing limited emergency administration

High-risk infrastructure changes should still be performed only when an appropriate recovery path exists.

---

# Operational Risk

Remote access does not make every remote change safe.

This became particularly important during a Proxmox cluster upgrade.

HomeProx hosts:

- Active routing infrastructure
- DNS
- The Tailscale subnet router

Rebooting this host remotely could simultaneously interrupt:

- Internet routing
- DNS
- Management-network access
- Tailscale subnet routing

If the host failed to return after maintenance, remote access could be lost.

For that reason, disruptive maintenance on HomeProx was intentionally postponed until physical recovery access was available.

This demonstrates an important systems-administration principle:

> Remote connectivity is not a substitute for a reliable recovery path.

---

# High-Availability Considerations

The current subnet-router architecture introduces an availability dependency on HomeProx.

If HomeProx is unavailable, remote subnet access may also become unavailable.

Future improvements may evaluate:

- Additional subnet routers
- Tailscale high-availability routing
- Alternative always-on subnet-router placement
- Integration with the firewall architecture
- Improved recovery procedures

Any redundant configuration must be tested before being relied upon during maintenance.

---

# DNS Considerations

The homelab uses Pi-hole for internal DNS filtering.

Remote administration therefore requires consideration of both:

- Network routing
- DNS resolution

A remote device may successfully reach an internal IP address while still failing to resolve an internal hostname.

Remote-access testing should validate both connectivity and DNS behavior.

---

# Troubleshooting

The Tailscale deployment has provided experience troubleshooting several networking issues.

Examples include:

- DNS resolution problems
- Linux IP forwarding
- Subnet route advertisement
- Route approval
- LXC TUN device permissions
- Tailscale service failures
- Network reachability
- Overlay versus local-network connectivity

Troubleshooting requires determining whether a failure exists at the:

- Client
- Tailscale overlay
- Subnet router
- Host firewall
- OPNsense firewall
- VLAN
- DNS
- Destination service

---

# Connectivity Validation

Remote connectivity can be tested at several layers.

## Tailscale Connectivity

Tailscale connectivity can be tested using:

```bash
tailscale ping <device>
```

This helps verify communication between Tailscale nodes.

---

## IP Connectivity

Internal network reachability can be tested using:

```bash
ping <internal-host>
```

This validates basic routed connectivity where ICMP is permitted.

---

## Service Connectivity

Testing the actual application port provides better validation than ping alone.

For example, from a Windows administrative workstation:

```powershell
Test-NetConnection <internal-host> -Port <service-port>
```

This verifies that the remote client can reach the intended service.

---

# Security Considerations

The Tailscale network should be treated as part of the administrative security boundary.

Important considerations include:

- Authorizing only trusted devices
- Removing unused devices
- Restricting unnecessary subnet access
- Protecting administrative accounts
- Maintaining endpoint security
- Reviewing access policies
- Keeping Tailscale updated
- Monitoring remote-access availability
- Avoiding unnecessary public management ports
- Maintaining OPNsense firewall segmentation

Remote access should follow the principle of least privilege wherever practical.

---

# Monitoring

Tailscale availability is planned for inclusion in the centralized homelab monitoring environment.

Potential monitoring checks include:

- Tailscale service availability
- Subnet-router availability
- Route availability
- Remote management reachability
- Unexpected node disconnections
- DNS availability through remote access

---

# Exception-Based Health Reporting

The future automated health-reporting system should report Tailscale problems when they affect administrative availability.

Example:

```text
REMOTE ACCESS
Status: WARNING

Tailscale subnet router:
Unavailable

Management subnet:
Not reachable through remote-access path

Recommended action:
Verify Tailscale service and HomeProx connectivity.
```

Normal connectivity should not generate unnecessary alerts.

---

# Future Improvements

Planned improvements include:

- Complete remote testing from an external network
- Validate laptop access away from home Wi-Fi
- Validate iPhone access over cellular
- Verify internal DNS resolution remotely
- Review Tailscale access-control policies
- Evaluate redundant subnet routing
- Monitor subnet-router availability
- Integrate Tailscale status into Grafana
- Integrate remote-access failures into health reports
- Document emergency remote-access procedures

---

# Lessons Learned

## Secure Access Does Not Require Public Management Interfaces

Overlay networking can provide remote access without directly publishing infrastructure administration interfaces to the Internet.

## Routing and DNS Are Separate Problems

Successful subnet routing does not automatically guarantee internal hostname resolution.

Both must be tested.

## The Subnet Router Is Infrastructure

Once a system provides remote access to other networks, its availability becomes part of the infrastructure dependency chain.

## Remote Maintenance Requires Recovery Planning

A remotely accessible system should not automatically be rebooted simply because remote access currently works.

The effect of that reboot on the remote-access path itself must be considered.

## Layered Security Still Matters

Tailscale complements rather than replaces:

- Firewalling
- VLAN segmentation
- Endpoint security
- Authentication
- Monitoring

---

# Skills Demonstrated

This project demonstrates practical experience with:

- Tailscale
- WireGuard-based networking
- Secure remote administration
- Linux networking
- IP forwarding
- Subnet routing
- Route advertisement
- VLAN networking
- OPNsense
- DNS troubleshooting
- Network troubleshooting
- Remote systems administration
- Access-control planning
- Network segmentation
- Risk assessment
- Infrastructure monitoring
- Technical documentation

---

# Project Status

**Operational / Validation In Progress**

Tailscale is deployed within the homelab, and HomeProx provides subnet routing for the management network.

Local connectivity testing has been performed.

Additional validation from external networks, including laptop and mobile-device testing, remains part of the project before the remote-access architecture is considered fully validated.

The documentation will be updated as testing, access-control improvements, monitoring, and redundancy are completed.
