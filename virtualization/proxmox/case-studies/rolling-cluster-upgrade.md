# Case Study: Rolling Proxmox VE Cluster Upgrade

## Executive Summary

This case study documents a rolling maintenance and upgrade procedure performed on my three-node Proxmox VE homelab cluster.
The objective was to update the Proxmox hosts while maintaining cluster quorum, protecting hosted workloads, minimizing service interruption, and verifying recovery options before making potentially disruptive changes.
During the maintenance process, I discovered that two important containers were not included in the scheduled backup job. Rather than proceeding with the upgrade, I stopped the maintenance process, created and verified manual backups, and then continued with the rolling upgrade.
Two nodes were successfully upgraded from an older Proxmox kernel to `7.0.14-20-pve`. The final node, which hosts critical routing, DNS, and remote-access infrastructure, was intentionally deferred because I was administering the environment remotely and could not guarantee recovery access if the node failed to restart.
This project demonstrates practical experience with Proxmox VE administration, Linux package management, cluster quorum, backup validation, risk assessment, change management, and post-maintenance verification.

---

## Environment

The homelab consists of a three-node Proxmox VE cluster.

| Node | Hardware | Primary Workloads |
|------|----------|-------------------|
| HomeProx | Dell OptiPlex 7020 Micro | OPNsense, Pi-hole, remote-access infrastructure |
| HomeProx2 | HP Elite Mini 600 G9 | AMP / Minecraft, NFS backup storage |
| HomeProx3 | HP Elite Mini 600 G9 | Prometheus, Grafana, Dashy |

The cluster uses Corosync for cluster communication and consists of three voting nodes.

With all nodes online:

- Expected votes: 3
- Total votes: 3
- Quorum requirement: 2
- Cluster state: Quorate

This architecture allows one Proxmox node to be offline for maintenance while the remaining two nodes maintain cluster quorum.

---

## Maintenance Objectives

The primary objectives of the maintenance were to:

1. Update all Proxmox hosts.
2. Install the latest available Proxmox kernel.
3. Maintain cluster quorum throughout the process.
4. Verify backups before rebooting workloads.
5. Minimize disruption to hosted services.
6. Validate each node before proceeding to the next.
7. Avoid making changes that could result in loss of remote administrative access.

A rolling maintenance strategy was selected rather than updating or rebooting multiple nodes simultaneously.

---

## Initial Cluster Health Verification

Before making changes, cluster health was verified using:

```bash
pvecm status
```

The cluster reported:

```text
Nodes:            3
Expected votes:   3
Total votes:      3
Quorum:           2
Quorate:          Yes
```

All three nodes were members of the cluster.

This confirmed that one node could safely be removed from service while the other two maintained quorum.

---

# Phase 1 — HomeProx2

HomeProx2 was selected as the first upgrade target.

The node hosted:

- AMP / Minecraft
- NFS backup storage
- A stopped standby OPNsense VM

The primary expected service impact was temporary loss of the game server during the eventual reboot.

---

## Package Assessment

Available package updates were reviewed before installation.

An APT distribution-upgrade simulation was performed:

```bash
apt-get -s dist-upgrade
```

The simulation reported:

```text
182 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
```

No packages were scheduled for removal.

This was especially important because unexpected removal of Proxmox packages would have caused the upgrade to be stopped and investigated before proceeding.

---

## Backup Investigation

Before upgrading the node, I inspected the existing Proxmox backup configuration.

The configured NFS backup storage was:

```text
nfs: infra-backups
        export /var/lib/vz/homeprox-backups
        path /mnt/pve/infra-backups
        server [backup-server]
        content backup
        nodes HomeProx
        prune-backups keep-all=1
```

This revealed that the shared backup storage was intentionally available to the primary HomeProx node rather than directly exposed as backup storage on every cluster node.

I then inspected the scheduled `vzdump` job.

The backup job contained:

```text
compress zstd
enabled 1
mode snapshot
schedule 3:00
storage infra-backups
vmid 100,101
```

This revealed an important problem.

The automated backup job included:

- VM 100 — OPNsense
- CT 101 — Pi-hole

It did **not** include:

- CT 102 — AMP / Minecraft
- CT 103 — Monitoring

This meant I could not assume the application workloads had usable recovery points.

---

## Backup Gap Discovery

I searched the existing backup directory for CT 102.

No existing backup was found.

Rather than continuing with the upgrade, I stopped the maintenance process and created a manual backup.

This was an important change-management decision: package installation was technically possible, but proceeding without a recovery point would have introduced unnecessary risk.

---

## Creating the AMP Backup

A snapshot-mode compressed backup was created:

```bash
vzdump 102 --storage local --mode snapshot --compress zstd
```

Proxmox:

1. Identified CT 102.
2. Created a temporary LVM snapshot.
3. Froze the guest filesystem.
4. Created the snapshot.
5. Thawed the filesystem.
6. Generated the compressed archive.
7. Removed the temporary snapshot.

The resulting archive was approximately:

```text
5.4 GB
```

The backup log was then checked rather than assuming the presence of a file meant the backup had succeeded.

The log reported:

```text
INFO: Finished Backup of VM 102 (00:00:28)
INFO: Backup job finished successfully
end task ... OK
```

The backup was therefore confirmed successful before maintenance continued.

---

## HomeProx2 Upgrade

The distribution upgrade was then performed:

```bash
apt dist-upgrade
```

The update installed new Proxmox components and generated an initramfs for:

```text
7.0.14-20-pve
```

Before rebooting, the installed Proxmox versions were checked:

```bash
pveversion -v
```

Relevant results included:

```text
proxmox-ve: 9.2.0
pve-manager: 9.2.21
proxmox-kernel-7.0: 7.0.14-20
proxmox-kernel-7.0.14-20-pve-signed: 7.0.14-20
```

The system was still running the previous kernel because a reboot had not yet occurred.

---

## HomeProx2 Reboot and Kernel Validation

HomeProx2 was rebooted.

After reconnecting, the running kernel was checked:

```bash
uname -r
```

Result:

```text
7.0.14-20-pve
```

This confirmed that HomeProx2 successfully booted into the new kernel.

---

## Cluster Validation

After the reboot, cluster status was checked again:

```bash
pvecm status
```

The cluster reported:

```text
Nodes:            3
Expected votes:   3
Total votes:      3
Quorum:           2
Quorate:          Yes
```

All three nodes were present.

This confirmed:

- HomeProx2 successfully rejoined Corosync.
- Cluster communication was operational.
- Full quorum had been restored.

---

## Application Validation

CT 102 was checked after the reboot:

```bash
pct status 102
```

The container reported:

```text
status: running
```

AMP/Minecraft had therefore returned to service successfully.

A final package check showed no remaining upgrades:

```bash
apt list --upgradable
```

No packages were returned.

HomeProx2 maintenance was considered complete.

---

# Phase 2 — HomeProx3

HomeProx3 was upgraded next.

This node hosts the monitoring environment, including:

- Prometheus
- Grafana
- Dashy

Before installing updates, its backup state was checked.

---

## Monitoring Backup Verification

Available local backups were listed:

```bash
pvesm list local --content backup
```

No backups were present.

Available storage was then checked:

```bash
pvesm status
```

The local storage had sufficient free capacity to create a backup.

Because CT 103 was not included in the automated backup schedule, I again stopped the upgrade process and created a recovery point.

---

## Creating the Monitoring Backup

CT 103 was backed up using:

```bash
vzdump 103 --storage local --mode snapshot --compress zstd
```

The backup completed with:

```text
INFO: Total bytes written: 4857681920
INFO: archive file size: 2.04GB
INFO: cleanup temporary 'vzdump' snapshot
INFO: Finished Backup of VM 103 (00:00:37)
INFO: Backup job finished successfully
```

This provided a verified recovery point before modifying the host.

---

## HomeProx3 Upgrade Simulation

The upgrade was simulated:

```bash
apt-get -s dist-upgrade
```

Result:

```text
183 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
```

No packages were scheduled for removal.

---

## HomeProx3 Upgrade

The distribution upgrade was performed:

```bash
apt dist-upgrade
```

After installation, the installed versions were checked.

Relevant results included:

```text
proxmox-ve: 9.2.0
pve-manager: 9.2.21
proxmox-kernel-7.0: 7.0.14-20
proxmox-kernel-7.0.14-20-pve-signed: 7.0.14-20
```

HomeProx3 was then rebooted.

---

## HomeProx3 Post-Upgrade Validation

After reboot:

```bash
uname -r
```

returned:

```text
7.0.14-20-pve
```

The new kernel was active.

Cluster health was checked:

```bash
pvecm status
```

The cluster again reported:

```text
Nodes:            3
Expected votes:   3
Total votes:      3
Quorum:           2
Quorate:          Yes
```

All three nodes had returned to cluster membership.

CT 103 was then checked and confirmed to be running.

A final package check:

```bash
apt list --upgradable
```

returned no remaining packages.

HomeProx3 maintenance was therefore considered complete.

---

# Phase 3 — HomeProx Risk Assessment

The final node required additional consideration.

HomeProx hosts several services that are critical to the operation and remote administration of the network:

- Active OPNsense router/firewall
- Pi-hole DNS
- Tailscale subnet router
- Primary infrastructure services

Unlike the other two nodes, rebooting HomeProx could interrupt the network itself.

---

## Backup Verification

The NFS backup repository was checked:

```bash
pvesm list infra-backups --content backup
```

Recent backups existed for both critical workloads.

The backup history contained multiple recovery points for:

### VM 100 — OPNsense

Multiple daily compressed `vma.zst` backups were available.

### CT 101 — Pi-hole

Multiple daily compressed `tar.zst` backups were available.

The latest backups had been created earlier the same day.

---

## Upgrade Simulation

HomeProx was also evaluated using:

```bash
apt-get -s dist-upgrade
```

Result:

```text
172 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
```

The upgrade plan itself appeared safe.

However, package safety was not the only consideration.

---

# Remote Maintenance Risk Decision

At the time of the maintenance, I was not physically at the homelab location.

HomeProx provides:

- Network routing
- Firewall services
- DNS
- Tailscale subnet routing
- Remote access to internal management networks

Although additional OPNsense virtual machines existed on other Proxmox nodes, they were not operating as a verified automatic failover system.

If HomeProx failed to boot correctly after the upgrade, I could potentially lose:

- Internet routing
- Internal DNS
- Management connectivity
- Tailscale access
- The ability to remotely repair the problem

For that reason, I deliberately postponed the final upgrade and reboot until physical access to the environment was available.

This was not a technical failure.

It was a deliberate risk-management decision.

> A change being technically possible does not mean it should be performed when a safe recovery path cannot be guaranteed.

---

# Issues Discovered During Maintenance

## 1. Incomplete Backup Coverage

The largest issue discovered during the maintenance process was incomplete automated backup coverage.

The scheduled backup job protected:

- OPNsense
- Pi-hole

but did not protect:

- AMP / Minecraft
- Monitoring infrastructure

Manual backups were created before continuing maintenance.

Expanding the automated backup policy was identified as a follow-up project.

---

## 2. Local-Only Recovery Points

The manually created backups for CT 102 and CT 103 were stored locally on their respective Proxmox nodes.

These backups protect against problems caused by software upgrades or configuration changes but do not protect against complete failure of the host's local storage.

A future improvement is to ensure all important workloads have automated off-node recovery points.

---

## 3. Router Dependency

The maintenance process highlighted the network's dependency on the active OPNsense VM hosted by HomeProx.

Standby OPNsense VMs exist, but automatic failover has not yet been validated.

Future work will evaluate router recovery and failover options.

---

# Rolling Maintenance Procedure Developed

Based on this maintenance session, the following procedure can be reused for future Proxmox upgrades:

### 1. Verify cluster health

```bash
pvecm status
```

Confirm:

- All expected nodes are present.
- Cluster is quorate.
- No unexpected Corosync problems exist.

### 2. Inventory workloads

```bash
qm list
pct list
```

Determine what services will be affected by maintenance.

### 3. Verify backups

Check that important workloads have recent recovery points.

Do not assume that a configured backup job includes every workload.

### 4. Create missing backups

Example:

```bash
vzdump <VMID> --storage local --mode snapshot --compress zstd
```

Verify successful completion.

### 5. Simulate the upgrade

```bash
apt-get -s dist-upgrade
```

Review:

- Upgraded packages
- New packages
- Removed packages

Investigate unexpected removals before continuing.

### 6. Install updates

```bash
apt dist-upgrade
```

### 7. Verify installed versions

```bash
pveversion -v
```

Confirm that the expected Proxmox packages and kernel were installed.

### 8. Reboot one node

Never reboot multiple cluster nodes simultaneously during routine rolling maintenance.

### 9. Verify active kernel

```bash
uname -r
```

### 10. Verify cluster membership

```bash
pvecm status
```

Confirm the node has returned and full quorum has been restored.

### 11. Verify workloads

Examples:

```bash
pct status <VMID>
qm status <VMID>
```

Confirm important services have returned.

### 12. Check remaining updates

```bash
apt list --upgradable
```

### 13. Continue to the next node

Only proceed after the current node has been completely validated.

---

# Results

Two of the three Proxmox nodes were successfully upgraded using a rolling maintenance strategy.

### HomeProx2

- Proxmox packages updated
- New kernel installed
- Successfully booted `7.0.14-20-pve`
- Rejoined cluster
- Cluster quorum restored
- AMP container returned to service
- No remaining package updates

### HomeProx3

- Proxmox packages updated
- New kernel installed
- Successfully booted `7.0.14-20-pve`
- Rejoined cluster
- Cluster quorum restored
- Monitoring container returned to service
- No remaining package updates

### HomeProx

- Critical backups verified
- Upgrade simulation completed successfully
- No package removals proposed
- Actual upgrade/reboot deliberately deferred until physical recovery access is available

---

# Follow-Up Improvements

The maintenance process produced several follow-up projects:

- Add CT 102 and CT 103 to automated backup coverage
- Store critical backups off-node
- Perform documented backup restoration tests
- Improve backup failure monitoring
- Add backup status to automated health reports
- Evaluate OPNsense recovery/failover
- Document disaster-recovery procedures
- Expand centralized logging
- Implement exception-based infrastructure health reporting

---

# Lessons Learned

## Verify backups instead of assuming they exist

A configured backup system does not guarantee every workload is protected.

The maintenance process directly identified two workloads missing from the automated backup schedule.

## Validate backups before making changes

The existence of an archive alone was not considered sufficient. Backup completion logs were checked for successful task completion.

## Quorum must be considered during cluster maintenance

A three-node cluster can tolerate one unavailable voting node while maintaining a two-vote quorum, making sequential rolling maintenance possible.

## Workload dependencies matter

A hypervisor reboot may affect much more than the hypervisor itself.

Understanding what each node hosts is necessary before deciding whether maintenance is safe.

## Remote access is part of the dependency chain

Rebooting the system responsible for routing and remote access while administering the environment remotely introduces a unique failure scenario.

## Post-change validation is part of the change

The upgrade was not considered successful merely because APT completed.

Success required verifying:

- Active kernel
- Cluster membership
- Quorum
- Workload state
- Remaining updates

---

# Skills Demonstrated

This project demonstrates practical experience with:

- Proxmox VE administration
- Linux systems administration
- Debian package management
- KVM/QEMU virtualization
- LXC containers
- Corosync cluster administration
- Cluster quorum
- Linux kernel upgrades
- Rolling maintenance
- NFS storage
- Backup administration
- Snapshot-based backups
- Disaster-recovery planning
- Service validation
- Infrastructure dependency analysis
- Troubleshooting
- Change management
- Risk assessment
- Remote systems administration
- Technical documentation

---

## Project Status

**Partially Complete**

HomeProx2 and HomeProx3 have been successfully upgraded and validated.

The final HomeProx upgrade is intentionally pending until physical access is available because the node hosts the active network gateway, DNS service, and remote-access infrastructure.

This case study will be updated after the final node maintenance is completed.
