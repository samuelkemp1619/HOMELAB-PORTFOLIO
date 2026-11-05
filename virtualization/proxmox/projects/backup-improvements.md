# Proxmox Backup Coverage Gap — Discovery and Mitigation

> **Completed historical case study:** This documents the discovery and immediate mitigation of a backup-coverage gap during maintenance of an earlier three-node Proxmox cluster. The workload placement and backup-job inventory below are historical, not a description of the current four-node environment. Successful backup tasks were verified; an isolated restore was not demonstrated in this case study.

## Project Overview

During maintenance of the earlier three-node Proxmox VE cluster, I reviewed scheduled backups before rolling host upgrades. The existing job protected primary network infrastructure workloads but omitted application and monitoring containers. I paused the upgrades and created manual recovery points for those workloads before proceeding.

**Scope of completed work:** Identifying the coverage gap, checking storage and backup configuration, creating manual backups, and confirming successful task logs. Subsequent scheduled backup configuration is documented separately in the [Proxmox overview](../README.md).

---

## Historical Environment

At the time of this incident, the Proxmox environment consisted of three physical cluster nodes hosting network infrastructure, application services, monitoring services, and backup storage.

| Node | Primary Workloads |
|------|-------------------|
| Node A | OPNsense, Pi-hole, remote-access infrastructure |
| Node B | AMP / Minecraft, backup storage |
| Node C | Prometheus, Grafana, Dashy |

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

## Backup Protection and Limitations at the Time

At the time of this incident, scheduled Proxmox backups protected primary infrastructure workloads using off-node storage. Manual local `vzdump` archives supplied immediate recovery points for the application and monitoring containers.

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


## Restore Validation Limitation

The task logs confirmed that backup jobs finished successfully. This does **not** establish that the archives could be restored, booted, or used to recover working applications. No isolated restore execution is documented by this case study.

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


These considerations explain the distinction between creating a recovery point and demonstrating a complete disaster-recovery capability; they are not claims that a recovery runbook was implemented or tested.

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

## Project Status

**Historical incident outcome: Completed.** The backup coverage gap was identified before maintenance, and manual recovery points were successfully created for the affected workloads. This completed the immediate mitigation documented in this case study. Subsequent scheduled backup configuration is documented separately. A successful isolated restore has not been demonstrated by this case study.
