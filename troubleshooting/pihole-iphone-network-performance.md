# Case Study: Investigating iPhone Network Performance with Pi-hole

## Executive Summary

This case study documents an investigation into poor network performance experienced by iPhones on a network using Pi-hole for centralized DNS filtering.

Because Pi-hole was part of the DNS path, it was initially a possible cause of the perceived performance problem. Rather than immediately changing DNS configuration, I tested the network systematically to determine whether the evidence pointed toward DNS, Pi-hole itself, the gateway, or the wireless/client path.

Testing showed that connectivity between the Pi-hole server and the network gateway was fast and stable, generally producing sub-millisecond latency with only small occasional increases.

Testing involving the affected iPhone showed significantly higher and more variable latency, with observed samples ranging from approximately 30 ms to over 200 ms.

Pi-hole was also confirmed to be operational, listening for DNS requests, blocking enabled, and configured with functioning upstream DNS resolvers.

The investigation therefore shifted away from treating Pi-hole as the primary suspect and toward the client/Wi-Fi network path.

A definitive root cause was not established during the investigation, so this case study documents the evidence and narrowed fault domain rather than claiming an unsupported resolution.

---

# Environment

The environment included:

- Pi-hole
- Linux
- iPhone clients
- Wireless network
- Local gateway/router
- Cloudflare upstream DNS
- Ethernet-connected infrastructure

Pi-hole provided centralized DNS resolution and filtering for network clients.

---

# Problem

iPhones connected to Wi-Fi were experiencing poor or inconsistent network performance.

Because DNS resolution passes through Pi-hole, one possible explanation was that Pi-hole or its upstream DNS configuration was delaying requests.

Possible causes included:

- Pi-hole performance
- DNS resolution delays
- Upstream DNS problems
- Local network latency
- Wi-Fi performance
- Client-specific behavior
- Gateway connectivity
- Packet loss
- Wireless interference

The objective was to gather evidence before changing infrastructure configuration.

---

# Troubleshooting Strategy

The investigation was divided into several layers:

```text
iPhone
   |
   v
Wi-Fi
   |
   v
Local Network
   |
   v
Gateway
   |
   v
Pi-hole / DNS
   |
   v
Upstream DNS
   |
   v
Internet
```

Testing individual portions of this path helps identify where abnormal behavior begins.

---

# Step 1 — Verify Pi-hole Operation

The first step was to confirm that Pi-hole itself was operating normally.

Checks confirmed that:

- Pi-hole was running.
- DNS blocking was enabled.
- DNS services were listening.
- IPv4 DNS was available.
- IPv6 DNS was available.
- Both UDP and TCP DNS were available.

This reduced the likelihood that the issue was caused by the DNS service simply being unavailable.

---

# Step 2 — Verify Upstream DNS

The Pi-hole configuration was reviewed to determine which upstream DNS resolvers were being used.

The configured upstream resolvers included Cloudflare DNS:

```text
1.1.1.1
1.0.0.1
```

The configuration therefore had valid upstream DNS servers available.

This did not completely eliminate DNS performance as a possible factor, but it confirmed that Pi-hole was not configured without usable upstream resolvers.

---

# Step 3 — Test Pi-hole to Gateway Connectivity

The next step was to determine whether the Pi-hole host itself had poor connectivity to the local gateway.

A multi-packet ping test was performed from the Pi-hole system to the gateway:

```bash
ping -c 20 <gateway>
```

Observed response times were generally very low.

Representative results included approximately:

```text
0.454 ms
0.600 ms
0.624 ms
0.548 ms
2.26 ms
0.775 ms
4.33 ms
```

Most responses were sub-millisecond, with only occasional small increases.

This suggested that the wired path between the Pi-hole system and the gateway was healthy.

---

# Finding: Pi-hole Network Path

The Pi-hole host did not demonstrate the large latency variation being observed on the affected mobile client.

The approximate comparison was:

```text
Pi-hole → Gateway

Typical:
< 1 ms

Occasional:
A few milliseconds
```

This was important because a severely overloaded or poorly connected Pi-hole host would be expected to show additional symptoms.

The available evidence did not indicate that behavior.

---

# Step 4 — Compare the iPhone Path

Latency involving the affected iPhone was significantly more variable.

Observed samples were approximately within the range:

```text
30 ms – 214 ms
```

This was substantially different from the wired infrastructure test.

Conceptually:

```text
Wired Pi-hole Path
<1 ms typical
     |
     | Stable
     v
Gateway


iPhone / Wireless Path
30–214+ ms observed
     |
     | Highly variable
     v
Network
```

The difference changed the direction of the investigation.

---

# Step 5 — Reevaluate the Initial Hypothesis

The initial hypothesis included Pi-hole as a possible performance bottleneck.

However, the evidence now showed:

### Pi-hole

- DNS service operational
- Blocking enabled
- Upstream DNS configured
- Stable local gateway connectivity
- Very low wired latency

### iPhone Path

- Significantly higher latency
- Large latency variation
- Problem associated with wireless client behavior

This made the wireless/client path a more useful area to investigate than immediately modifying the Pi-hole deployment.

---

# DNS vs. Network Latency

One of the important distinctions in this investigation was the difference between DNS latency and general network latency.

DNS primarily affects hostname resolution:

```text
example.com
    |
    v
DNS Query
    |
    v
IP Address
```

After resolution, the client communicates with the resulting IP address.

If the client demonstrates high latency to local network devices themselves, DNS alone cannot explain that latency.

This is why testing local IP connectivity was useful.

---

# Fault-Domain Isolation

The troubleshooting process progressively reduced the likely fault domain.

Initially:

```text
Possible Causes

├── Pi-hole
├── DNS
├── Upstream DNS
├── Router
├── Ethernet
├── Wi-Fi
└── iPhone
```

After testing:

```text
Pi-hole
   |
   └── Local wired connectivity appears healthy

Upstream DNS
   |
   └── Valid resolvers configured

Gateway path from Pi-hole
   |
   └── Low latency

Wireless / Client Path
   |
   └── High and variable latency observed
```

This does not prove that Wi-Fi was the final root cause.

It does show that further investigation should focus more heavily on the wireless/client path.

---

# Why Pi-hole Was Not Immediately Removed

A common troubleshooting reaction would be to bypass Pi-hole immediately.

That would introduce another variable before establishing whether the problem actually involved DNS.

Instead, the investigation first collected evidence from the existing configuration.

This reduced the risk of:

- Changing unrelated settings
- Introducing additional problems
- Losing the original test condition
- Incorrectly attributing improvement to DNS changes

---

# Further Investigation

Additional testing could include:

## Compare Multiple Wireless Devices

Determine whether the latency occurs on:

- One iPhone
- Multiple iPhones
- Other wireless devices

If only one device is affected, the client becomes a stronger suspect.

---

## Compare Wi-Fi and Ethernet

A wired client on the same network can provide a useful baseline.

```text
Wired Client
     vs.
Wireless Client
```

Large differences may indicate a wireless-specific problem.

---

## Test Gateway Latency from the Client

Repeated testing to the local gateway can help separate Internet performance from local Wi-Fi behavior.

If gateway latency is already high, the problem exists before Internet traffic or upstream DNS becomes relevant.

---

## Check Packet Loss

Latency alone does not provide the complete picture.

Testing should also evaluate:

- Packet loss
- Jitter
- Retransmissions

---

## Evaluate Wireless Conditions

Potential wireless factors include:

- Signal strength
- Channel congestion
- Interference
- Access-point placement
- Band selection
- Roaming behavior
- Power-saving behavior
- Client-specific wireless behavior

---

## Compare DNS Directly

DNS response time can also be measured independently.

Testing Pi-hole and an external resolver separately would help determine whether DNS resolution itself contributes meaningful delay.

The results should be compared rather than assuming one resolver is faster.

---

# Root Cause

**Not conclusively determined.**

The available evidence did not support identifying Pi-hole as the confirmed root cause.

The strongest observed difference was between the stable wired infrastructure path and the much more variable latency associated with the iPhone/wireless path.

Further wireless and client testing would be required before assigning a definitive root cause.

---

# Resolution Status

**Investigation narrowed — additional testing required.**

The troubleshooting process successfully narrowed the problem domain from the entire DNS/network infrastructure toward the wireless/client side of the connection.

No unnecessary production DNS changes were made solely on the basis of assumption.

---

# Lessons Learned

## Do Not Assume DNS Is Responsible for Every Slow Connection

Because Pi-hole participates in DNS resolution, it is easy to blame it when browsing feels slow.

Network latency should be tested independently.

## Establish a Baseline

The wired Pi-hole-to-gateway test provided a baseline against which the iPhone results could be compared.

## Compare Multiple Network Paths

Testing from only the affected device would not have shown whether the infrastructure itself had similar latency.

## Local Testing Matters

Testing the local gateway helps remove Internet routing and remote servers from the troubleshooting equation.

## Avoid Premature Configuration Changes

Changing DNS servers before collecting evidence could obscure the actual problem.

## A Narrowed Fault Domain Is a Valid Troubleshooting Result

Not every investigation immediately produces a definitive root cause.

Reducing the number of likely causes is still meaningful technical progress.

## Document Uncertainty

When evidence does not establish a root cause, documentation should state that explicitly rather than presenting a hypothesis as fact.

---

# Skills Demonstrated

This investigation demonstrates experience with:

- Network troubleshooting
- DNS troubleshooting
- Pi-hole administration
- Linux networking
- ICMP testing
- Latency analysis
- Fault-domain isolation
- Client/server troubleshooting
- Wireless troubleshooting methodology
- Upstream DNS configuration
- Evidence-based troubleshooting
- Root-cause analysis
- Technical documentation

---

# Project Status

**Investigation Narrowed / Root Cause Not Yet Confirmed**

Pi-hole and its wired network path showed healthy local connectivity during testing.

The affected iPhone demonstrated significantly higher and more variable latency.

Further client and wireless-network testing is required before identifying a definitive root cause.
