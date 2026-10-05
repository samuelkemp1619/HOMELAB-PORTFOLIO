# Proxmox Backup & Recovery Improvements

## Project Overview

During routine maintenance of my three-node Proxmox VE cluster, I reviewed the existing backup configuration before performing rolling host upgrades.

This review identified a gap in automated backup coverage. The scheduled backup job protected the primary network infrastructure workloads, but some application and monitoring workloads were not included.

Rather than continuing maintenance without verified recovery points, I paused the upgrade process and created manual backups of the affected containers.

This project documents the identified backup gap, the immediate mitigation performed, and the planned improvements to the homelab backup and disaster-recovery strategy.

---

## Environment

The Proxmox environment consists of three physical cluster nodes hosting network infrastructure, application services, monitoring services, and backup storage.

| Node | Primary Workloads |
|------|-------------------|
| HomeProx | OPNsense, Pi-hole, remote-access infrastructure |
| HomeProx2 | AMP / Minecraft, backup storage |
| HomeProx3 | Prometheus, Grafana, Dashy |

Proxmox `vzdump` is used to create compressed VM and LXC backups.

---

## Problem Identified

Before performing cluster maintenance, I reviewed the existing scheduled Proxmox backup configuration.

The automated backup job included:

- OPNsense
- Pi-hole

However, it did **not** include:

- AMP / Minecraft
- Monitoring infrastructure

This created inconsistent recovery coverage across the environment.

Critical network infrastructure had scheduled recovery points, while other important workloads depended on the continued availability of their local Proxmox storage.

---

## How the Gap Was Discovered

The issue was identified as part of the pre-maintenance validation process rather than after a failure.

Before upgrading each Proxmox node, I checked:

1. Cluster health
2. Hosted workloads
3. Available backups
4. Available storage
5. Package upgrade plans
6. Recovery options

When the existing backup configuration was inspected, only the OPNsense and Pi-hole workloads were included in the scheduled backup job.

The missing workloads were therefore identified before their hosts were modified or rebooted.

---

# Immediate Mitigation

I decided not to continue the host upgrades until recovery points existed for the affected workloads.

Manual snapshot-mode backups were created using Proxmox `vzdump`.

General command:

```bash
vzdump <VMID> --storage local --mode snapshot --compress zstd
```

Snapshot mode allowed the containers to be backed up with minimal interruption while Zstandard compression reduced archive size.

---

## AMP / Minecraft Backup

The AMP / Minecraft container was manually backed up before upgrading its Proxmox host.

The backup command followed the format:

```bash
vzdump <VMID> --storage local --mode snapshot --compress zstd
```

The resulting compressed archive was approximately:

```text
5.4 GB
```

The backup task log was checked and confirmed:

```text
INFO: Backup job finished successfully
```

The host upgrade was allowed to continue only after the recovery point had been verified.

---

## Monitoring Infrastructure Backup

The monitoring container hosting services such as Prometheus, Grafana, and Dashy was also missing from the scheduled backup job.

Before upgrading its Proxmox host, I verified that sufficient local storage was available and created another snapshot-mode backup.

The resulting compressed archive was approximately:

```text
2.04 GB
```

The task log again reported:

```text
INFO: Backup job finished successfully
```

This provided a recovery point for the monitoring environment before host maintenance continued.

---

# Current Backup Architecture

The environment currently uses Proxmox backup jobs and `vzdump` archives for workload protection.

The primary infrastructure workloads already have scheduled backups stored away from the node hosting those workloads.

Manual local backups were used as an immediate safety measure for the application and monitoring containers.

This created an important distinction between two types of protection.

### Off-Node Backup

A backup stored on another physical system can protect against:

- VM or container corruption
- Failed updates
- Configuration errors
- Hypervisor failure
- Local storage failure
- Failure of the original physical host

### Local Backup

A backup stored on the same physical host can protect against:

- Failed application updates
- Configuration errors
- VM/container corruption
- Unsuccessful maintenance

However, it does **not** adequately protect against complete failure of the host or its storage.

For this reason, the manually created local backups are considered a temporary mitigation rather than the final backup design.

---

# Remaining Risk

The current backup strategy still requires improvement.

Important remaining risks include:

- Some recovery points are stored locally.
- Automated backup coverage is not yet consistent across all important workloads.
- Restore testing has not yet been fully documented.
- Backup failure monitoring can be improved.
- Backup age and recovery-point availability should be monitored automatically.
- Recovery procedures should be documented before an actual emergency occurs.

---

# Remediation Plan

The permanent backup strategy will be developed in several stages.

## 1. Inventory Workloads

All Proxmox VMs and containers will be reviewed and classified.

Possible classifications include:

### Critical Infrastructure

Examples:

- Router/firewall
- DNS
- Authentication
- Remote-access infrastructure

### Important Services

Examples:

- Monitoring
- Application servers
- Game/application management services

### Lab / Rebuildable Systems

Systems that can be recreated quickly from documentation may require different retention policies than critical infrastructure.

---

## 2. Expand Automated Backup Coverage

Important workloads currently relying on manual backups should be added to scheduled backup jobs.

The objective is to eliminate dependence on remembering to manually create recovery points before routine maintenance.

---

## 3. Improve Off-Node Protection

Important workloads should have recovery points stored on a physical system other than the system hosting the original workload.

This protects against failure of:

- Local disks
- Filesystems
- Proxmox nodes
- Host hardware

---

## 4. Establish Retention Policies

Backup retention should balance recovery flexibility with available storage capacity.

Retention may include combinations of:

- Recent backups
- Daily backups
- Weekly backups
- Monthly backups

Critical infrastructure may require different retention policies than easily rebuilt lab systems.

---

## 5. Monitor Backup Health

Backup jobs should eventually be incorporated into centralized monitoring.

The monitoring environment should detect conditions such as:

- Failed backup jobs
- Missing expected backups
- Backup repository unavailable
- Backup older than the expected recovery window
- Backup storage approaching capacity
- Unexpected changes in backup size
- Restore-test failures

Successful routine backups should not generate unnecessary alerts unless a summary is specifically requested.

---

# Restore Testing

A successful backup job does not necessarily prove that a system can be recovered.

For that reason, the final backup strategy should include documented restoration testing.

A restore test should validate more than archive extraction.

The process should verify:

1. The backup archive can be read.
2. The VM or container can be restored.
3. The restored workload can boot.
4. Network configuration is correct.
5. Required application services start.
6. Application data is present.
7. Clients can reach the restored service.
8. No production workload is accidentally affected during testing.

---

## Example Restore Validation

A future restore test may use an isolated VM/container ID and isolated networking.

The restored workload should not automatically be connected to the production network if doing so could create duplicate IP addresses, DNS services, gateways, or other conflicts.

This allows recovery procedures to be tested without disrupting the production homelab.

---

# Monitoring Integration

Backup health is planned for integration with the centralized Prometheus/Grafana monitoring environment and the future exception-based health reporting system.

The goal is not simply to display another dashboard.

Instead, the system should identify conditions that require attention.

For example:

```text
BACKUPS
Status: WARNING

Monitoring container:
Expected backup not found within recovery window.

Last successful backup:
Outside configured threshold.

Recommended action:
Verify scheduled Proxmox backup job.
```

Normal backup operation should generally remain quiet.

---

# Exception-Based Health Reporting

The future homelab health-reporting system will include backup status as one of its infrastructure checks.

Potential report categories include:

- Proxmox cluster health
- Service availability
- Network health
- DNS health
- Tailscale connectivity
- Backup status
- Resource anomalies
- Log anomalies
- Security events

Backup reporting should focus on exceptions such as failures or missing recovery points rather than listing every successful backup operation.

---

# Security Considerations

Backups can contain complete copies of systems and therefore may contain sensitive configuration data.

Backup infrastructure should be treated as security-sensitive.

Important controls include:

- Restricting access to backup storage
- Limiting unnecessary network exposure
- Protecting administrative credentials
- Maintaining appropriate filesystem permissions
- Monitoring unauthorized access
- Protecting backup integrity
- Maintaining appropriate retention policies
- Controlling who can perform restores

A compromised backup repository could provide an attacker with data that is not directly accessible from the production service.

---

# Disaster-Recovery Considerations

Backup creation is only one component of disaster recovery.

A complete recovery strategy must also answer questions such as:

- Where are the backups stored?
- How quickly can they be accessed?
- Which workloads must be restored first?
- What infrastructure must exist before restoration?
- How will networking be restored?
- How will DNS be restored?
- How will the router/firewall be restored?
- What happens if an entire Proxmox node fails?
- What happens if the backup server fails?
- Can the environment be recovered without Internet access?

These questions will be addressed as the disaster-recovery documentation is expanded.

---

# Lessons Learned

## Backup Configuration Must Be Verified

Having a scheduled backup job does not mean every important workload is protected.

The actual workload list must be reviewed.

## Recovery Points Should Exist Before Maintenance

Creating a recovery point after something fails is too late.

Backup verification should be part of the pre-change process.

## Local Backups Are Not Enough

A backup stored on the same physical host provides useful protection but does not protect against complete host or storage failure.

## Backup Success Is Not Restore Success

A backup job reporting success does not prove that the workload can be restored and operated correctly.

Restore testing is necessary.

## Maintenance Can Reveal Infrastructure Weaknesses

The backup gap was discovered because the environment was reviewed before a routine upgrade.

Maintenance procedures can therefore serve as an opportunity to identify weaknesses before they become incidents.

---

# Skills Demonstrated

This project demonstrates experience with:

- Proxmox VE administration
- Linux systems administration
- LXC backup administration
- VM backup administration
- Proxmox `vzdump`
- Snapshot-based backups
- Zstandard compression
- Backup verification
- Storage management
- NFS storage
- Disaster-recovery planning
- Recovery-point management
- Risk assessment
- Change management
- Infrastructure monitoring
- Troubleshooting
- Technical documentation

---

# Future Improvements

Planned improvements include:

- Add missing workloads to scheduled backup coverage
- Improve off-node backup coverage
- Review backup retention policies
- Monitor backup failures
- Monitor backup age
- Monitor backup storage capacity
- Perform isolated restore testing
- Document recovery procedures
- Integrate backup health into Grafana
- Integrate backup exceptions into automated health reports
- Develop a complete disaster-recovery runbook

---

# Project Status

**Remediation Pending**

The backup coverage gap has been identified.

Temporary recovery points were successfully created for the affected workloads before host maintenance was performed.

The next phase will implement permanent automated backup coverage, improve off-node protection, and perform documented restore testing.

This document will be updated with the final configuration, validation results, and recovery-test evidence as those improvements are implemented.
