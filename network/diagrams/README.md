# Homelab Network Architecture

**Updated:** October 26, 2026

This is a **logical topology**, not a verified port-by-port cable map. It documents network segmentation, cluster placement, and service relationships. Physical uplinks and upstream WAN details are intentionally simplified. Dotted arrows indicate logical dependencies rather than physical cabling.

## Current Logical Network Diagram

```mermaid
flowchart TB
    WAN((Internet)) --> EDGE["WAN / upstream gateway"]
    EDGE --> FW["OPNsense virtual firewall / router<br/>HomeProx • VM 100"]
    FW --> SWITCH["Managed switching<br/>VLAN-aware infrastructure"]
    SWITCH --> V10["VLAN 10 • Management"]
    SWITCH --> V20["VLAN 20 • Trusted"]
    SWITCH --> V30["VLAN 30 • IoT"]
    SWITCH --> V40["VLAN 40 • Guest"]
    V10 --> CLUSTER["Proxmox VE • four-node cluster"]
    CLUSTER --> P1["HomeProx<br/>Dell OptiPlex Micro"]
    CLUSTER --> P2["HomeProx2<br/>HP Elite Mini"]
    CLUSTER --> P3["HomeProx3<br/>HP Elite Mini"]
    CLUSTER --> P4["HomeProx4<br/>Dell PowerEdge R640"]
    P1 --> DNS1["CT 101 • Primary Pi-hole"]
    P2 --> APP["CT 102 • AMP"]
    P2 --> OBS["CT 103 • Dashy / Prometheus / Grafana<br/>Nebula Sync"]
    P3 --> VAULT["CT 104 • Vaultwarden"]
    P3 --> DNS2["CT 105 • Secondary Pi-hole"]
    P4 --> WIN["VM 400 • Windows Server 2025"]
    P4 --> AI["AI-Operations / n8n<br/>scheduled alerting paused"]
    DNS1 <-. "config sync" .-> DNS2
    FW -. "DHCP advertises both resolvers" .-> DNS1
    FW -. "DHCP advertises both resolvers" .-> DNS2
    V20 --> GRAPHY["Graphy • Raspberry Pi 4<br/>Grafana kiosk"]
    GRAPHY -. "dashboard display" .-> OBS
    REMOTE["Remote administration • Tailscale"] -. "encrypted access" .-> CLUSTER
    CLUSTER -. "scheduled backups" .-> BACKUP["Off-node backup storage"]
    classDef core fill:#183153,color:#fff,stroke:#4c8ad5
    classDef network fill:#e5f0ff,color:#17365d,stroke:#77a5d6
    classDef service fill:#e9f7f1,color:#174d36,stroke:#71b49a
    class FW,CLUSTER core
    class SWITCH,V10,V20,V30,V40 network
    class DNS1,DNS2,OBS,VAULT,GRAPHY,APP,WIN,AI service
```

[Open standalone Mermaid source](homelab-network.mmd)

## Segmentation and routing

| Network | Purpose | Documentation status |
|---|---|---|
| VLAN 10 | Infrastructure management | In use |
| VLAN 20 | Trusted clients, including Graphy | In use |
| VLAN 30 | IoT segment | Configured; DNS enforcement client validation pending |
| VLAN 40 | Guest segment | Configured; DNS enforcement client validation pending |

OPNsense handles inter-network policy. The primary and secondary Pi-hole instances are advertised as DNS resolvers; their independent operation and Nebula Sync synchronization have been tested, but automatic client failover is **not verified**. Direct-DNS bypass rules were configured for four networks; comprehensive client testing and DNS-over-HTTPS controls remain outstanding.

## Four-node virtualization cluster

- **HomeProx:** OPNsense VM 100 and primary Pi-hole CT 101.
- **HomeProx2:** AMP CT 102 and monitoring/application stack CT 103 (Dashy, Prometheus, Grafana, exporters, Nebula Sync).
- **HomeProx3:** Vaultwarden CT 104 and secondary Pi-hole CT 105.
- **HomeProx4:** Windows Server 2025 VM 400 and AI-Operations tooling.

Proxmox scheduled backups use off-node storage. The n8n alert workflow has been configured and manually tested, but scheduled production execution remains unpublished/paused.

## Access and monitoring

Tailscale enables remote administrative access. Graphy is a Raspberry Pi 4 kiosk on the trusted network displaying Grafana dashboards; it is a display endpoint, not a monitoring server.

## Scope and limitations

This diagram does **not** assert verified switch-port assignments, redundant physical network paths, OPNsense HA, DNS client failover, or completed Windows Active Directory deployment. It intentionally omits private IPs, credentials, and firewall rule details. No hardware inspection details are included.
