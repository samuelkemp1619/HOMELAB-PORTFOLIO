# Case Study: Troubleshooting Tailscale in a Proxmox LXC Environment

## Executive Summary

This case study documents troubleshooting performed while deploying and operating Tailscale within my Proxmox homelab.

Tailscale was being used to provide secure remote connectivity between infrastructure systems. During deployment, I encountered two different types of problems:

1. DNS resolution problems after Tailscale modified DNS behavior.
2. Tailscale service failures inside an LXC container related to access to the Linux TUN device.

The investigation required distinguishing between DNS failures, general network connectivity, the Tailscale service itself, and limitations introduced by container virtualization.

The troubleshooting reinforced an important networking principle:

> A successful overlay-network deployment depends on the operating system, virtualized environment, routing, DNS, and underlying network all functioning together.

---

# Environment

The environment included:

- Proxmox VE
- Linux
- LXC containers
- Tailscale
- Pi-hole
- Internal DNS
- Multiple Proxmox hosts
- Virtual machines and containers
- Segmented homelab networking

Tailscale was being evaluated and deployed as part of the secure remote-administration architecture.

---

# Objective

The objective was to provide secure connectivity to homelab infrastructure without exposing administrative interfaces directly to the public Internet.

The desired architecture was:

```text
Remote Device
      |
      v
Tailscale
      |
      v
Homelab Infrastructure
      |
      +---- Proxmox
      |
      +---- Containers
      |
      +---- Monitoring
      |
      +---- Internal Services
```

Tailscale uses WireGuard-based encrypted networking to create an overlay network between authorized systems.

---

# Problem 1 — DNS Resolution Failure

During the Tailscale deployment, DNS configuration became an early troubleshooting issue.

After DNS behavior was changed to use Tailscale-related DNS configuration, general name resolution stopped functioning correctly in part of the environment.

This produced a situation where network connectivity and DNS functionality had to be investigated separately.

---

# Understanding the Failure

Network connectivity and DNS resolution are different functions.

A system may be able to reach:

```text
1.1.1.1
```

while failing to resolve:

```text
example.com
```

In that scenario:

```text
IP Connectivity = Working

DNS Resolution = Failing
```

Testing only hostnames could therefore make the entire network appear unavailable even though basic IP connectivity remained operational.

---

# DNS Investigation

The resolver configuration was reviewed to determine which DNS service the system was attempting to use.

The troubleshooting process included checking:

- `/etc/resolv.conf`
- Current DNS resolver configuration
- Tailscale DNS behavior
- General Internet connectivity
- Hostname resolution

The investigation determined that the DNS configuration introduced during the Tailscale setup was interfering with normal resolution.

---

# DNS Remediation

General DNS functionality was restored by returning the system to a working resolver configuration rather than continuing to rely on the problematic configuration.

After the change, DNS resolution was retested.

This restored normal name resolution and allowed the Tailscale investigation to continue independently of the DNS issue.

---

# Lesson from the DNS Failure

This incident demonstrated why DNS should be tested separately from network connectivity.

A useful troubleshooting sequence is:

```text
Can I reach the gateway?
        |
        v
Can I reach an Internet IP?
        |
        v
Can I resolve a hostname?
        |
        v
Can I reach the application?
```

Each test validates a different layer.

---

# Problem 2 — Tailscale Service Failure in LXC

A separate problem occurred when Tailscale was used inside a Proxmox LXC container.

The Tailscale service failed because the container did not have the required access to the Linux TUN device.

This was different from the earlier DNS problem.

The service itself could not establish the networking interface required for normal operation.

---

# What Is `/dev/net/tun`?

Linux uses the TUN/TAP subsystem to provide virtual network interfaces to userspace applications.

The TUN device is commonly available through:

```text
/dev/net/tun
```

Software such as VPN and overlay-networking applications can use this interface to create virtual Layer 3 network devices.

Conceptually:

```text
Tailscale
    |
    v
/dev/net/tun
    |
    v
Virtual Network Interface
    |
    v
Encrypted Overlay Network
```

If an application cannot access the required TUN device, it may be unable to create the network interface it needs.

---

# Why LXC Matters

An LXC container does not behave exactly like a full virtual machine.

Containers share the host's Linux kernel while receiving isolated userspace environments.

Conceptually:

```text
Proxmox Host
      |
      | Linux Kernel
      |
   -----------
   |         |
   v         v
 LXC 1     LXC 2
```

Because containers share the host kernel, access to devices and kernel capabilities can be restricted.

This improves isolation but can affect applications requiring special networking capabilities.

---

# Fault Domain

The service failure introduced several possible causes:

```text
Tailscale Service
       |
       +---- Configuration
       |
       +---- Authentication
       |
       +---- DNS
       |
       +---- Network Connectivity
       |
       +---- Linux Permissions
       |
       +---- TUN Device
       |
       +---- LXC Restrictions
```

The investigation therefore needed to determine whether the problem existed inside Tailscale configuration or at the virtualization boundary.

---

# Service Investigation

The Tailscale service state was examined using Linux service-management tools.

Typical checks for this type of problem include:

```bash
systemctl status tailscaled
```

and:

```bash
journalctl -u tailscaled
```

Service logs are important because a failed networking service may provide more useful information than simply testing connectivity.

The failure indicated a problem associated with TUN access rather than ordinary Internet connectivity.

---

# Device Investigation

The next layer was the TUN device.

A Linux system normally exposes:

```text
/dev/net/tun
```

for applications requiring TUN networking.

Inside a restricted LXC container, however, the device may not automatically be available or permitted.

This shifted the investigation toward Proxmox container configuration.

---

# Proxmox LXC Device Access

For an LXC workload to use certain host devices, the container may require explicit permission to access those devices.

The general architecture becomes:

```text
Proxmox Host
      |
      v
/dev/net/tun
      |
 Container Permission
      |
      v
LXC Container
      |
      v
tailscaled
```

If the device exists on the host but is unavailable inside the container, Tailscale cannot necessarily use it in the normal way.

---

# Troubleshooting Approach

The investigation followed several layers.

## Layer 1 — Underlying Network

Verify that the container can reach the local network and Internet independently of Tailscale.

Example:

```bash
ping <gateway>
```

and:

```bash
ping 1.1.1.1
```

---

## Layer 2 — DNS

Verify hostname resolution separately:

```bash
getent hosts example.com
```

or:

```bash
nslookup example.com
```

This determines whether a failure is caused by DNS rather than the overlay network.

---

## Layer 3 — Tailscale Service

Check:

```bash
systemctl status tailscaled
```

Then inspect logs:

```bash
journalctl -u tailscaled
```

---

## Layer 4 — TUN Device

Verify whether the TUN device exists:

```bash
ls -l /dev/net/tun
```

The result should be evaluated both on the Proxmox host and, where appropriate, inside the container.

---

## Layer 5 — Container Configuration

If the device exists on the host but not inside the container, inspect the Proxmox LXC configuration.

This helps determine whether device access or container permissions are preventing Tailscale from operating.

---

# DNS and TUN Were Separate Problems

An important part of this troubleshooting process was recognizing that the two failures were not necessarily the same problem.

The environment experienced:

```text
Problem A
Tailscale-related DNS configuration
        |
        v
Hostname resolution problems
```

and separately:

```text
Problem B
LXC device restrictions
        |
        v
TUN unavailable
        |
        v
Tailscale service failure
```

Treating them as one issue could have resulted in repeatedly changing DNS settings while the service failure was actually occurring at the container/device layer.

---

# Host vs. Container Deployment

The troubleshooting also highlighted an architectural decision when deploying networking software.

## Installing Tailscale on the Host

Advantages can include:

- Direct access to host networking
- Fewer container device restrictions
- Easier subnet routing
- Reduced dependence on LXC device passthrough

However, software installed directly on the hypervisor should be kept minimal.

---

## Installing Tailscale in LXC

Advantages include:

- Workload isolation
- Independent configuration
- Easier application-level backup
- Reduced software installed directly on the hypervisor

Potential complications include:

- TUN permissions
- Container capabilities
- Device access
- Additional routing complexity

The deployment location should therefore be selected according to the role the Tailscale node will perform.

---

# Subnet Routing

The final remote-access architecture uses a Tailscale subnet router to provide access to internal management infrastructure.

Conceptually:

```text
Remote Device
      |
      v
Tailscale
      |
      v
Subnet Router
      |
      v
Management Network
      |
      +---- Proxmox
      |
      +---- Monitoring
      |
      +---- Internal Services
```

This means every internal service does not need its own independent Tailscale installation.

---

# IP Forwarding

A subnet router must forward packets between networks.

On Linux, this requires IP forwarding.

The persistent forwarding configuration in the environment is maintained using:

```text
/etc/sysctl.d/99-tailscale-forwarding.conf
```

This ensures the required forwarding behavior survives reboot.

---

# Validation

Tailscale troubleshooting should validate multiple layers rather than relying on a single connectivity test.

## Overlay Connectivity

```bash
tailscale ping <device>
```

This verifies communication through the Tailscale network.

---

## Internal IP Connectivity

```bash
ping <internal-host>
```

This helps validate subnet routing where ICMP is allowed.

---

## Application Connectivity

From a Windows administrative device:

```powershell
Test-NetConnection <internal-host> -Port <service-port>
```

This verifies that the actual application port can be reached.

---

## DNS Resolution

Hostname resolution should be tested independently.

This prevents successful IP routing from being confused with successful DNS operation.

---

# Current Architecture

The current homelab design uses Tailscale as part of the secure remote-administration architecture.

A Proxmox host provides subnet routing for the management network.

This avoids requiring every infrastructure workload to operate as an independent Tailscale endpoint.

Remote-access validation and resiliency improvements remain ongoing.

---

# Security Considerations

Tailscale provides encrypted connectivity, but it does not eliminate the need for other security controls.

The environment should continue to use:

- OPNsense firewall policy
- VLAN segmentation
- Strong authentication
- Trusted administrative endpoints
- Endpoint security
- Restricted management access
- Monitoring

The Tailscale network itself should also be treated as a privileged administrative network.

---

# Lessons Learned

## DNS and Connectivity Must Be Tested Separately

A DNS failure can make a functioning network appear unavailable.

Testing raw IP connectivity helps separate the two.

## Containers Share the Host Kernel

LXC workloads may require explicit access to devices or capabilities that would normally be available in a full virtual machine.

## Read the Service Logs

Connectivity tests cannot explain why a service failed to start.

`systemctl` and `journalctl` provide evidence from the application itself.

## Understand the Virtualization Boundary

A problem inside a container may actually originate from restrictions imposed by the host.

## Similar Symptoms Can Have Different Causes

DNS failure and Tailscale service failure both affected connectivity, but they occurred at different layers.

## Deployment Architecture Matters

Installing networking software directly on a host versus inside a container introduces different security, management, and compatibility considerations.

## Validate the Actual Application

Successful ping testing does not prove that an administrative web interface or service port is reachable.

---

# Skills Demonstrated

This troubleshooting project demonstrates experience with:

- Proxmox VE
- Linux
- LXC
- Tailscale
- WireGuard-based networking
- Linux TUN/TAP
- `/dev/net/tun`
- Linux service management
- `systemd`
- `journalctl`
- DNS troubleshooting
- Linux networking
- IP forwarding
- Subnet routing
- Overlay networking
- Network troubleshooting
- Virtualization troubleshooting
- Fault-domain isolation
- Secure remote administration
- Technical documentation

---

# Future Improvements

Planned improvements include:

- Complete remote validation from external networks
- Validate mobile access over cellular
- Validate laptop access away from the home network
- Verify internal DNS resolution remotely
- Review Tailscale access policies
- Monitor Tailscale service health
- Monitor subnet-router availability
- Evaluate subnet-router redundancy
- Document emergency remote-access recovery
- Integrate remote-access health into centralized monitoring

---

# Project Status

**Operational / Additional Validation Planned**

The original DNS issue was corrected by restoring functional DNS resolution.

The LXC troubleshooting identified TUN-device access as a separate virtualization-related requirement for Tailscale deployments inside containers.

The current remote-access architecture uses Tailscale subnet routing, while additional external-network validation and resiliency work remain planned.
