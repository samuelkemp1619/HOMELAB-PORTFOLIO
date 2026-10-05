# Case Study: Centralized Homelab Monitoring & Observability

## Executive Summary

This case study documents the deployment of a centralized monitoring environment for my Proxmox homelab.

A dedicated Linux container hosts Prometheus, Grafana, and Dashy, providing a centralized platform for infrastructure monitoring, visualization, and service access.

The monitoring environment is designed around a larger objective: move away from manually checking individual systems and toward centralized, exception-based infrastructure monitoring.

The current implementation provides the foundation for future centralized logging, automated health reporting, anomaly detection, and critical-service alerting.

---

# Environment

The monitoring environment runs as a dedicated LXC container within a three-node Proxmox VE cluster.

| Component | Role |
|---|---|
| Proxmox VE | Virtualization platform |
| LXC | Monitoring workload isolation |
| Prometheus | Metrics collection |
| Grafana | Metrics visualization |
| Dashy | Centralized service dashboard |
| Raspberry Pi | Dedicated monitoring display |

The monitoring container is hosted on HomeProx3.

Separating monitoring into its own container allows the monitoring environment to be independently:

- Updated
- Backed up
- Restarted
- Troubleshot
- Restored
- Expanded

---

# Problem

As the homelab expanded, manually checking individual systems became increasingly inefficient.

The environment contains multiple infrastructure components, including:

- Proxmox hosts
- Virtual machines
- LXC containers
- OPNsense
- Pi-hole
- Network services
- Application services
- Backup infrastructure
- Remote-access infrastructure

Without centralized monitoring, determining overall infrastructure health would require logging into multiple systems individually.

The objective was therefore to create a centralized monitoring platform capable of answering:

> Is the infrastructure operating normally, and if not, what requires attention?

---

# Solution

A dedicated monitoring container was deployed on HomeProx3.

The logical architecture is:

```text
Infrastructure
      |
      | Metrics
      v
  Prometheus
      |
      v
   Grafana
      |
      v
 Visualization

Infrastructure Services
      |
      v
    Dashy
      |
      v
Centralized Service Access
```

Prometheus and Grafana provide the monitoring foundation while Dashy provides a centralized interface for accessing homelab services.

---

# Prometheus

Prometheus provides time-series metrics collection.

Rather than relying entirely on point-in-time observations, historical metrics make it possible to investigate behavior over time.

This is useful for identifying conditions such as:

- Sustained resource utilization
- Service interruptions
- Availability changes
- Network abnormalities
- Performance degradation
- Infrastructure trends

Prometheus also provides the metrics foundation required for future automated reporting and alerting.

---

# Grafana

Grafana provides visualization of monitoring data.

Dashboards allow infrastructure information to be reviewed from a centralized location rather than requiring administrators to inspect each system independently.

Grafana is intended to provide visibility into areas such as:

- Infrastructure health
- Service availability
- Host behavior
- Resource utilization
- Historical trends
- Network behavior

The objective is not simply to create large numbers of graphs.

Dashboards should help answer operational questions and support troubleshooting.

---

# Dashy

Dashy provides a centralized interface for homelab services.

It complements the monitoring stack by providing quick access to administrative and monitoring interfaces.

Conceptually:

```text
Administrator
      |
      v
    Dashy
      |
      +---- Proxmox
      |
      +---- Grafana
      |
      +---- Pi-hole
      |
      +---- Other Services
```

This reduces the need to maintain separate bookmarks or manually remember service locations.

---

# Dedicated Monitoring Container

The monitoring stack is intentionally separated from the Proxmox hosts themselves.

This provides several advantages.

## Isolation

Monitoring applications do not need to be installed directly on the hypervisor operating system.

## Maintainability

Monitoring services can be updated or restarted without modifying the Proxmox host.

## Backup

The entire monitoring environment can be protected through Proxmox container backups.

## Recovery

The monitoring environment can potentially be restored as a complete workload rather than rebuilding each monitoring application independently.

---

# Monitoring Display

A Raspberry Pi is used as a dedicated physical monitoring display.

The device operates as a kiosk-style endpoint for displaying Grafana or other monitoring dashboards.

The architecture is:

```text
Prometheus
    |
    v
 Grafana
    |
    v
Network
    |
    v
Raspberry Pi
    |
    v
Dedicated Display
```

The Raspberry Pi is a display endpoint rather than the monitoring backend itself.

This provides a persistent infrastructure-status display without dedicating a full workstation to the task.

---

# Monitoring Philosophy

A major design goal is to avoid creating monitoring that generates excessive noise.

For example, continuously reporting:

```text
CPU: 14%
Memory: 37%
Disk: 42%
```

provides little operational value when those values are normal.

Instead, routine monitoring should emphasize exceptions.

Examples include:

```text
CPU usage above expected threshold for an extended period

Memory pressure affecting a service

Storage approaching capacity

Unexpected service outage

DNS resolution failures

Backup failure

Cluster quorum problem

Abnormal network latency
```

This creates a distinction between:

**visibility** — information available when investigating a system

and:

**attention** — conditions that should actively be surfaced to an administrator.

---

# Exception-Based Monitoring

The long-term monitoring architecture will focus on conditions that deviate from normal operation.

An abnormal resource condition should consider both:

- Magnitude
- Duration

For example, a short CPU spike may be normal.

Sustained high CPU utilization may require investigation.

Similarly:

```text
CPU 95% for 3 seconds
```

may not justify an alert, while:

```text
CPU 95% for 20 minutes
```

could indicate a problem.

This approach helps reduce alert fatigue.

---

# Monitoring Categories

The environment is being designed to monitor several infrastructure categories.

## Proxmox

Potential checks include:

- Node availability
- Cluster membership
- Quorum
- VM/container state
- Storage availability
- Abnormal resource utilization

## Network

Potential checks include:

- Gateway availability
- Packet loss
- Latency
- Critical interface state
- Network-service availability

## DNS

Potential checks include:

- Pi-hole availability
- DNS resolution
- Upstream resolver failures
- Abnormal DNS behavior

## Remote Access

Potential checks include:

- Tailscale service availability
- Subnet-router availability
- Management-network reachability

## Backups

Potential checks include:

- Backup failures
- Missing expected recovery points
- Backup age
- Backup repository availability
- Backup storage capacity

## Applications

Potential checks include:

- Service availability
- Unexpected service termination
- Application-specific health indicators

---

# Backup Discovery During Maintenance

The monitoring environment itself demonstrated why infrastructure monitoring and operational validation are important.

During a rolling Proxmox upgrade, the backup configuration was reviewed before the HomeProx3 host was modified.

The monitoring container was discovered to be missing from the scheduled backup job.

Rather than continuing maintenance without a recovery point, a manual Proxmox snapshot backup was created.

The resulting compressed archive was approximately:

```text
2.04 GB
```

The backup log confirmed:

```text
INFO: Backup job finished successfully
```

Only after the backup was verified did host maintenance continue.

This identified automated backup coverage as an area requiring improvement.

---

# Post-Maintenance Validation

After HomeProx3 was upgraded and rebooted, several checks were performed.

The active Proxmox kernel was verified:

```bash
uname -r
```

The cluster was checked:

```bash
pvecm status
```

The monitoring container was then confirmed to have returned to a running state.

Finally:

```bash
apt list --upgradable
```

was used to confirm that the host had no remaining package updates.

This demonstrated that monitoring infrastructure should itself be included in change-management and recovery planning.

---

# Centralized Logging — Planned

Metrics provide only part of the information required to understand infrastructure behavior.

A system may report normal CPU, memory, and storage utilization while its logs contain important warnings or failures.

For that reason, centralized logging is planned as the next major observability component.

Grafana Loki is the preferred platform being considered.

The proposed architecture is:

```text
Proxmox --------\
Linux -----------\
OPNsense ----------> Centralized Logs
Pi-hole ----------/         |
Services --------/          v
                           Loki
                             |
                             v
                          Grafana
```

This would allow metrics and logs to be investigated through the same general observability environment.

---

# Log Analysis — Planned

The objective is not simply to collect every log entry.

The system should identify noteworthy events and reduce repetitive noise.

Potential event classifications include:

- Critical
- Warning
- Security
- Network
- Service
- Backup
- Hardware

For example, hundreds of identical low-priority warnings should not overwhelm a health report.

Instead, they could be summarized:

```text
WARNING

Repeated service warning detected
Occurrences: 347
Systems affected: 1
First observed: 02:14
Last observed: 07:51

Recommended action:
Review service configuration.
```

---

# Automated Health Reporting — Planned

A future reporting service is planned for the monitoring container.

The reporting system will combine:

```text
Prometheus Metrics
        +
Centralized Logs
        |
        v
Health Analysis
        |
        v
HTML Email Report
```

The report should answer:

> What happened in the homelab that requires attention?

rather than:

> What are all of the current metrics?

---

# Planned Health Report

Potential report sections include:

```text
HOMELAB HEALTH REPORT

Overall Status

Proxmox Cluster
Network
Critical Services
DNS
Remote Access
Backups
Resource Anomalies
Log Anomalies
Security Events
Recommended Attention
```

Normal metrics should generally be omitted unless they provide useful context.

---

# Example Exception Report

A future report might contain:

```text
OVERALL STATUS: WARNING

PROXMOX
All cluster nodes online.
Quorum healthy.

NETWORK
No critical gateway failures detected.

DNS
Pi-hole operational.

BACKUPS
WARNING:
Expected backup missing for one workload.

RESOURCE ANOMALIES
No sustained abnormal resource conditions detected.

LOG EVENTS
WARNING:
Repeated service errors detected on one system.

RECOMMENDED ATTENTION

1. Investigate missing backup.
2. Review repeated service errors.
```

This provides significantly more operational value than emailing a large collection of normal graphs.

---

# Immediate Alerting — Planned

Some events should not wait for a scheduled report.

Potential immediate alerts include:

- Loss of Proxmox quorum
- Gateway failure
- DNS outage
- Critical service failure
- Backup infrastructure failure
- Storage critically full
- Serious security event

Lower-severity events can remain in the scheduled report.

---

# Baseline & Anomaly Detection — Future

Static thresholds cannot identify every unusual condition.

Future monitoring may establish baseline behavior and identify deviations from normal operation.

For example:

```text
Normal DNS Queries:
10,000–15,000 per day

Observed:
85,000 queries

Status:
Anomalous
```

or:

```text
Normal Backup Duration:
4–6 minutes

Observed:
38 minutes

Status:
Investigate
```

This would allow the monitoring environment to become more context-aware over time.

---

# Security Monitoring

The observability platform can also support cybersecurity projects.

Future security-related monitoring may include:

- Authentication failures
- Unusual DNS activity
- Unexpected network connections
- Firewall events
- IDS/IPS alerts
- Honeypot activity
- Repeated service failures
- Unauthorized access attempts

This allows the monitoring platform to support both systems administration and cybersecurity experimentation.

---

# Monitoring Architecture Roadmap

The long-term architecture is:

```text
                  Homelab Infrastructure
                          |
             ---------------------------
             |                         |
          Metrics                     Logs
             |                         |
             v                         v
        Prometheus                   Loki
             |                         |
             -----------+-------------
                        |
                        v
                     Grafana
                        |
              -------------------
              |                 |
              v                 v
          Dashboards      Health Analysis
                                |
                    ------------------------
                    |                      |
                    v                      v
             Scheduled Report       Critical Alert
```

---

# Security Considerations

Monitoring infrastructure contains valuable information about the environment.

Access should therefore be controlled appropriately.

Security considerations include:

- Restrict administrative access
- Avoid unnecessary Internet exposure
- Protect monitoring credentials
- Maintain software updates
- Control log access
- Sanitize exported data
- Maintain backups
- Apply least privilege

Monitoring data published in this repository should also be sanitized before being made public.

---

# Lessons Learned

## Monitoring Must Include Monitoring

The monitoring platform itself requires backups, updates, availability checks, and recovery planning.

## Metrics Are Not Enough

Metrics can show that something changed, while logs may explain why it changed.

Both are useful for troubleshooting.

## More Alerts Do Not Mean Better Monitoring

Excessive alerts create noise and can cause important events to be ignored.

## Duration Matters

Temporary resource spikes are often normal.

Sustained abnormal behavior is generally more meaningful.

## Backups Should Be Monitored

A backup strategy should detect missing or failed recovery points rather than relying on manual inspection.

## Dashboards and Alerts Serve Different Purposes

Dashboards provide visibility.

Alerts identify conditions requiring attention.

## Observability Can Support Security

Centralized metrics and logs provide useful data for both infrastructure administration and cybersecurity analysis.

---

# Skills Demonstrated

This project demonstrates practical experience with:

- Prometheus
- Grafana
- Dashy
- Proxmox VE
- Linux
- LXC
- Infrastructure monitoring
- Time-series metrics
- Dashboard administration
- Service monitoring
- Backup validation
- Change management
- Observability architecture
- Alert design
- Log-management planning
- Anomaly-detection concepts
- Network monitoring
- Security monitoring
- Technical documentation

---

# Future Improvements

Planned improvements include:

- Deploy Grafana Loki
- Centralize infrastructure logs
- Expand Prometheus monitoring coverage
- Monitor Proxmox quorum
- Monitor backup health
- Monitor Pi-hole/DNS health
- Monitor Tailscale availability
- Add network latency and packet-loss monitoring
- Implement exception-based health reporting
- Generate scheduled HTML email reports
- Implement immediate critical alerts
- Develop baseline-based anomaly detection
- Add security-event monitoring
- Perform monitoring-container restore testing
- Add sanitized Grafana screenshots to the portfolio

---

# Project Status

**Operational / Expanding**

Prometheus, Grafana, and Dashy are operational on the dedicated monitoring container.

The current environment provides centralized monitoring and dashboard capabilities.

Centralized logging, exception-based reporting, advanced alerting, and anomaly detection remain planned improvements and are intentionally documented as future work rather than completed capabilities.
