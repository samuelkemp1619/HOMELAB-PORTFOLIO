# Troubleshooting & Incident Case Studies

## Overview

This section documents real troubleshooting scenarios encountered while building and operating my homelab.

The purpose is to demonstrate the troubleshooting process rather than simply documenting the final solution.

Each case study focuses on:

- Identifying symptoms
- Gathering evidence
- Developing hypotheses
- Testing individual infrastructure layers
- Eliminating possible causes
- Identifying root causes where possible
- Validating changes
- Documenting lessons learned

---

# Troubleshooting Methodology

When troubleshooting infrastructure problems, I generally work through the environment systematically rather than immediately changing configuration.

The basic process is:

```text
Observe Symptoms
       |
       v
Gather Evidence
       |
       v
Identify Possible Causes
       |
       v
Test Individual Layers
       |
       v
Eliminate Causes
       |
       v
Implement Change
       |
       v
Validate Result
       |
       v
Document Findings
```

---

## 1. Define the Symptom

Before changing configuration, determine exactly what is failing.

Examples include:

- DNS resolution failure
- High network latency
- Service unavailable
- VM/container unreachable
- Cluster communication failure
- Backup failure
- Remote-access failure

A clear symptom helps prevent unrelated configuration changes.

---

## 2. Determine the Scope

Determine whether the problem affects:

- One device
- One VLAN
- One application
- One server
- Multiple clients
- The entire network

Scope can significantly reduce the number of possible causes.

For example, if only one wireless client experiences poor performance while wired infrastructure remains healthy, the investigation should consider client or wireless behavior before assuming a core network failure.

---

## 3. Test from Multiple Points

Testing from different systems helps determine where a failure begins.

Examples include:

```text
Client → Gateway

Client → DNS

Server → Gateway

Server → Internet

Client → Internal Service

Remote Client → Tailscale → Internal Service
```

Comparing results helps isolate the failing layer.

---

## 4. Separate DNS from Connectivity

DNS problems can appear to be general network failures.

Whenever possible, test both:

```bash
ping <IP-address>
```

and:

```bash
nslookup <hostname>
```

If IP connectivity succeeds while hostname resolution fails, the problem is likely related to DNS rather than basic routing.

---

## 5. Validate the Actual Service

Ping alone does not prove an application is working.

Service ports should also be tested.

Example from Windows:

```powershell
Test-NetConnection <host> -Port <port>
```

This distinguishes basic host reachability from application availability.

---

## 6. Change One Variable at a Time

Changing multiple settings simultaneously makes it difficult to determine which change actually affected the problem.

Whenever practical:

1. Record the current state.
2. Make one controlled change.
3. Retest.
4. Compare results.
5. Continue only if necessary.

---

## 7. Preserve Evidence

Logs, command output, screenshots, timestamps, and monitoring data should be collected before restarting or modifying systems when practical.

Evidence may disappear after:

- Reboots
- Service restarts
- Log rotation
- Configuration changes

Preserving evidence improves root-cause analysis.

---

# Case Studies

This section will contain troubleshooting scenarios involving areas such as:

### DNS & Networking

- Pi-hole troubleshooting
- DNS resolution failures
- Client network-performance issues
- VLAN connectivity
- Firewall policy
- Network latency

### Proxmox

- Cluster communication
- Corosync
- Quorum
- LXC networking
- Backup failures
- Storage availability

### Remote Access

- Tailscale connectivity
- Subnet routing
- DNS through remote connections
- Linux TUN permissions

### Monitoring

- Prometheus collection
- Grafana dashboards
- Service availability
- Monitoring-container recovery

### Applications

- AMP / Minecraft
- Containerized services
- Dashboard integrations
- API connectivity

---

# Case Study Format

Individual troubleshooting case studies will generally follow this format:

```text
Problem

Environment

Symptoms

Initial Hypotheses

Evidence Collected

Testing

Findings

Resolution

Validation

Root Cause

Lessons Learned

Skills Demonstrated
```

If a root cause was not conclusively identified, the documentation will state that rather than presenting an assumption as fact.

---

# Security & Privacy

Troubleshooting evidence may contain sensitive information.

Before being published, information will be reviewed for:

- Passwords
- API keys
- Authentication tokens
- Private keys
- Email addresses
- Personally identifiable information
- Sensitive logs
- Public IP addresses
- Internal details that do not need to be disclosed

Commands and outputs may be sanitized while preserving the technical value of the investigation.

---

# Portfolio Objective

These case studies demonstrate more than familiarity with specific products.

They demonstrate the ability to:

- Approach technical problems systematically
- Gather useful evidence
- Work across multiple infrastructure layers
- Distinguish symptoms from root causes
- Validate assumptions
- Avoid unnecessary configuration changes
- Document technical findings
- Communicate troubleshooting processes clearly

These skills apply directly to network administration, systems administration, help desk escalation, cybersecurity operations, and infrastructure engineering.
