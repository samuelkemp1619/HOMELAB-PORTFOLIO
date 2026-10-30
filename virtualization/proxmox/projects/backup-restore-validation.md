# Proxmox Backup and Restore Validation — Test Plan and Evidence Log

**Project status:** Runbook documented; restore execution **not yet performed**  
**Environment:** Four-node Proxmox VE 9.x cluster  
**Scope:** Guest backup coverage, recoverability, service validation, and disaster-recovery readiness  
**Public documentation:** Generic node aliases; no private addresses, credentials, serials, or unique device identifiers

## Purpose and Success Criteria

Successful backup tasks demonstrate that archives were created, but they do not prove all guests are covered or that a restored guest will boot and serve its application. This project separates four questions:

1. **Coverage:** Is every in-scope guest assigned a backup policy?
2. **Freshness:** Is a recent successful recovery point available?
3. **Integrity:** Can the archive be read and restored?
4. **Functionality:** Does the recovered guest boot, contain expected data, and pass application checks?

A recovery test passes only after all four are documented for the selected guest. No pass is claimed in this initial runbook.

## Known Baseline (October 2026)

| Item | Documented baseline | Validation boundary |
|---|---|---|
| Scheduled backup jobs | Snapshot job at 03:00 and stop-mode job at 03:30 | Job existence does not imply full guest coverage |
| Compression | ZSTD | Restore still required |
| Storage | Off-node backup destination | Availability and failure domain should be checked at test time |
| Retention | Keep latest 3, daily 7, weekly 4, monthly 3 | Actual archive inventory must be checked |
| Notifications | Failure notification configured | Delivery and response process not proven |
| Backup logs | Successful tasks reviewed | Only reviewed workloads and runs are evidenced |
| Secondary DNS guest | Previously not included in reviewed scheduled job inventory | Confirm current policy before claiming protection |
| Restore exercise | No completed, documented isolated restore | **Pending** |

The historical [backup improvements project](backup-improvements.md) describes an earlier three-node configuration and manual pre-upgrade backups; it must not be mistaken for the current four-node inventory.

## Phase 1 — Capture Current Guest and Backup Inventory

Before any changes, collect sanitized output from the Proxmox management interface or CLI. Record each guest's public alias, owning node, business/service criticality, expected recovery-point objective, and backup coverage.

| Guest / service class | Importance | Scheduled coverage | Last successful archive | Restore priority |
|---|---|---|---|---|
| Virtual firewall | Critical | Verify | Verify | High |
| Primary DNS | Critical | Verify | Verify | High |
| Secondary DNS | Critical | **Coverage gap previously observed; recheck** | Verify | High |
| Monitoring stack | Important | Verify | Verify | Medium |
| Password-management service | Critical/sensitive | Verify | Verify | High |
| Application/game management | Important | Verify | Verify | Medium |
| Windows Server lab | Lab | Verify | Verify | Low |
| AI-Operations lab | Lab | Verify | Verify | Low |
| Email-organization utility | Important/sensitive | Verify | Verify | Medium |

Do not publish real guest IDs, backup archive names, internal IP addresses, or application secrets in evidence screenshots.

Suggested inspection commands (execute only when ready to validate):

```bash
pvesh get /cluster/backup --output-format json
pvesh get /cluster/resources --type vm --output-format json
pvesm status
```

Check backup archives using the relevant Proxmox storage view or `pvesm list <BACKUP_STORAGE_ID> --content backup`. Compare the actual archive inventory against all in-scope guests. Backup jobs may overlap; do not infer coverage from a single job's guest list.

## Phase 2 — Select a Safe Restore Candidate

Choose a noncritical, low-risk guest with a recent archive, small storage footprint, and a testable service. **Do not start by restoring the firewall, primary DNS, password vault, or an email-integrated workload.**

Prerequisites:

- Confirm the archive is from the intended guest and storage.
- Confirm sufficient target disk capacity and compatible guest storage.
- Reserve an unused temporary guest ID.
- Prepare an isolated network with **no production connectivity** or duplicate IP/gateway/DNS presence.
- Record the original guest's state and ensure the original will not be overwritten.
- Agree on a rollback and cleanup plan.

## Phase 3 — Isolated Restore Procedure (Not Yet Executed)

### A. Preflight

Record archive date, backup mode, storage destination, guest type (VM or LXC), and expected service behavior. Confirm the test target is isolated. Do not reuse a production guest ID.

### B. Restore

Use the Proxmox GUI's **Restore** action or the matching CLI, substituting actual verified identifiers at execution time:

For an LXC container:

```bash
pct restore <UNUSED_TEST_CTID> <VERIFIED_BACKUP_ARCHIVE> --storage <TEST_STORAGE>
```

For a virtual machine:

```bash
qmrestore <VERIFIED_BACKUP_ARCHIVE> <UNUSED_TEST_VMID> --storage <TEST_STORAGE>
```

These are **templates**, not commands already executed. Before first boot, inspect the restored guest configuration and disconnect or isolate its network interface. The safest isolation method depends on guest type and bridge configuration; verify it rather than assuming a particular flag works in every environment.

### C. Validate in Isolation

- Archive restore task completes without errors.
- Guest starts and reaches expected console/login state.
- Virtual disks, filesystems, and expected data are present.
- Required services start and pass local checks.
- No production IP, DNS, DHCP, routing, or application endpoint collision occurs.
- Restore duration and archive age are recorded.
- Application owner confirms the result where relevant.

### D. Cleanup

Stop and remove **only the temporary restored guest**, after verifying its ID and that it contains no production changes. Preserve sanitized test notes and task results. Do not delete the source archive or original workload.

## Phase 4 — Record Evidence

Complete this table during the real test. Leave fields **Not tested** until observed.

| Evidence item | Result |
|---|---|
| Test date/time | Not tested |
| Candidate guest alias | Not selected |
| Original host / restore host | Not selected |
| Backup archive timestamp | Not recorded |
| Backup storage / test storage | Not recorded |
| Restore task status | Not tested |
| Guest boot result | Not tested |
| Data verification | Not tested |
| Application health | Not tested |
| Network isolation check | Not tested |
| Restore duration (RTO observation) | Not measured |
| Archive age (RPO observation) | Not measured |
| Cleanup verification | Not tested |
| Overall decision | **Pending** |

**Terminology:** Observed restore duration and archive age are measurements from the test; they do not by themselves establish a guaranteed recovery time objective (RTO) or recovery point objective (RPO).

## Failure Handling and Stop Conditions

Stop and reassess if the test risks modifying the original guest, connecting a duplicate service to production, exposing sensitive backup data, exhausting storage, or disrupting critical DNS/firewall functions. Do not force a restore into production as a shortcut to validation.

A failed restore is a useful finding: record the failing stage, exact error (with secrets redacted), likely contributing factors, corrective action, and retest outcome.

## Prioritized Remediation Backlog

1. Reconcile all guest IDs against the **current** scheduled jobs and archive inventory.
2. Add or confirm backup coverage for the secondary Pi-hole and any other unprotected guests.
3. Select and restore a low-risk guest in isolation.
4. Validate data and service health; record recovery duration and archive age.
5. Test a representative higher-priority workload only after the first test procedure is proven safe.
6. Add alerting for missing/stale archives, backup job failures, and storage capacity.
7. Consider backup repository failure-domain and offsite protection requirements.

## Portfolio Outcome

**Completed:** Documented a structured backup coverage audit, isolated restore runbook, evidence matrix, and remediation plan.

**Not completed:** Live guest inventory reconciliation, a demonstrated restore, production recovery timing, and disaster-recovery certification.

## Related Projects

- [Proxmox cluster overview](../README.md)
- [Historical backup improvements](backup-improvements.md)
- [Cluster operations case study](../case-studies/four-node-cluster-operations.md)
- [Power-loss recovery incident](../case-studies/power-loss-recovery-incident.md)
