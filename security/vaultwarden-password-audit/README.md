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

Optional password-grouping enhancements were removed from the project backlog. Actual credential remediation is a separate activity and is not claimed as completed here.
