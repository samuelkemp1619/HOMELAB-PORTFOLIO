# Vaultwarden Password Audit — PowerShell Automation

**Status:** Completed — October 2026  
**Category:** Security Auditing / PowerShell / Data Analysis

## Overview

Developed and ran a PowerShell-based audit against an exported Vaultwarden login dataset to identify password hygiene issues for follow-up review. The work produced a reusable audit script and CSV results.

**Scope:** Inventory and analysis only. Completing the audit does **not** mean every identified password was changed or remediated.

## Audit Results

| Finding | Count |
|---|---:|
| Login entries analyzed | 573 |
| Entries using reused passwords | 480 |
| Potential duplicate entries | 10 |
| Entries missing website information | 7 |

These categories can overlap and should not be summed as distinct affected accounts.

## Completed Work

- [x] Built a working PowerShell audit script
- [x] Processed 573 login entries
- [x] Identified password reuse
- [x] Flagged potential duplicates
- [x] Flagged missing website fields
- [x] Exported and backed up audit results as CSV

## Approach

1. Read the exported login dataset locally.
2. Analyze relevant fields for reuse and data-quality issues.
3. Produce counts and structured CSV findings.
4. Review findings without exposing credentials in public documentation.

## Security and Privacy

**Do not commit** the original password export, plaintext passwords, credential hashes, account identifiers, or the unsanitized audit CSV. This portfolio entry intentionally documents methodology and aggregate findings only.

## Skills Demonstrated

PowerShell scripting, security hygiene assessment, structured data analysis, CSV reporting, finding classification, and sensitive-data handling.

## Scope Boundary

Optional password-grouping enhancements were removed from the project backlog. Actual credential remediation was a separate activity from this October audit; see the November 2026 hardening record below. The October audit results remain a historical baseline, not a verified post-rotation audit.

## November 2026 Homelab Credential Hardening

**Status:** Completed based on operator confirmation; individual application coverage was not independently audited.  
**Scope:** Homelab management accounts, not every entry in the 573-entry Vaultwarden audit.

### Completed activities

- Rotated root passwords individually on all four Proxmox hosts, using unique credentials; the first host's fresh web login was explicitly verified during the work, and the operator subsequently confirmed completion on all nodes.
- Rotated other homelab account passwords, including hardware-management credentials, according to operator confirmation.
- Enrolled Proxmox `root@pam` in TOTP MFA and verified a fresh login. The Proxmox user authentication configuration is cluster-wide.
- Generated and securely retained Proxmox recovery codes.
- Configured an OPNsense local TOTP authentication server, confirmed the underlying local-password test, verified TOTP authentication, and tested a successful web login after enabling MFA. In this configuration the working combined input was **TOTP code followed by password**.
- Confirmed Vaultwarden two-step login and a recovery code stored independently of the vault.
- Reviewed iDRAC9 separately: its password was rotated, but native TOTP MFA was **not** verified or claimed.

### Validation and limitations

This activity is separate from the October 2026 audit of 573 entries. No new full-vault scan was performed to establish that every historical reuse finding was resolved. MFA availability and coverage for every ancillary application were not individually validated. No secrets, OTP seeds, recovery codes, account-specific credentials, or internal network addresses are included in this public record.

### Skills demonstrated

Privileged account management, credential rotation, TOTP enrollment and troubleshooting, authentication-server testing, recovery planning, change sequencing, and access validation.
