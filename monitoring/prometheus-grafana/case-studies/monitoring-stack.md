# Case Study: Centralized Homelab Monitoring & Observability

## Executive Summary

This case study documents the completed deployment of a centralized monitoring environment for a four-node Proxmox VE homelab.

A dedicated Linux container on Node B hosts Prometheus, Grafana, and Dashy. Prometheus collects infrastructure metrics, Grafana provides visualization, and Dashy provides centralized access to homelab services.

A Raspberry Pi operates as a dedicated Grafana monitoring display.

The completed implementation provides centralized metrics collection, dashboard visualization, and service access. Automated exception reporting, centralized log aggregation, and advanced alerting are not demonstrated capabilities of this deployment.

---

# Environment

The monitoring environment runs as a dedicated LXC container within a four-node Proxmox VE cluster.

| Component | Role |
|---|---|
| Proxmox VE | Virtualization platform |
| LXC | Monitoring workload isolation |
| Prometheus | Metrics collection |
| Grafana | Metrics visualization |
| Dashy | Centralized service dashboard |
| Raspberry Pi | Dedicated monitoring display |

The monitoring container is hosted on Node B.

Separating monitoring into its own container supports independent updates, backups, restarts, troubleshooting, and maintenance. A successful manual backup was documented, but restoration has not been demonstrated.

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

A dedicated monitoring container is deployed on Node B.

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

# Monitoring Design Principles

The monitoring environment was designed to improve infrastructure visibility without requiring administrators to access each system individually.

Prometheus and Grafana provide access to metrics and dashboards that support troubleshooting and operational reviews.

An important distinction is between **visibility**, which provides information for investigation, and **alerting**, which actively notifies an administrator when intervention may be required.

This deployment demonstrates centralized visibility. Automated exception-based alerting and anomaly detection have not been implemented or validated as part of this case study.

---

# Backup Discovery During Maintenance

The monitoring environment itself demonstrated why infrastructure monitoring and operational validation are important.

During a rolling Proxmox upgrade, the backup configuration was reviewed before the Node C host was modified.

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

After Node C was upgraded and rebooted, several checks were performed.

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

- Prometheus metrics collection
- Grafana dashboard administration
- Dashy service dashboard configuration
- Proxmox VE and LXC administration
- Linux service management
- Infrastructure monitoring
- Time-series metrics visualization
- Monitoring architecture documentation
- Raspberry Pi kiosk deployment
- Backup coverage review
- Manual backup verification
- Post-maintenance service validation
- Change management
- Technical troubleshooting and documentation

These skills reflect completed deployment, maintenance, and validation activities rather than planned monitoring capabilities.

---

# Project Status

**Completed — Deployed Monitoring Stack**

Prometheus, Grafana, and Dashy were deployed within a dedicated Proxmox LXC container on Node B.

The environment provides centralized infrastructure metrics, dashboard visualization, and access to homelab services. A Raspberry Pi provides a dedicated monitoring display.

The monitoring container was included in a successful manual backup after a scheduled-backup coverage gap was identified.

**Validation limitations:**

- Successful backup creation was verified; restoration was not tested.
- Comprehensive monitoring coverage across all infrastructure components has not been established.
- Automated exception reporting, centralized logging, advanced alerting, and anomaly detection are not claimed as deployed capabilities.

The documented monitoring deployment is complete within this defined scope.
