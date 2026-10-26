# AI-Operations Proxmox Monitoring

**Status:** Manually tested; notification workflow configuration complete; scheduled activation pending (October 22, 2026).

## Environment
Debian VM on HomeProx4, Docker Compose, n8n, PostgreSQL, SearXNG, OpenAI GPT-5 mini. Four Proxmox nodes and nine guests monitored through Proxmox API.

## Implementation
- Combined node and VM/LXC health into structured JavaScript monitoring output.
- Added AI-generated health summaries and JSON Schema output validation.
- Built independent deterministic alert classification and IF routing.
- Recognized intentionally stopped OPNsense VMs 200 and 300 as expected.
- Configured Pushover API request, credentials, message and priority.
- Configured but did not activate an hourly trigger.

## Alert rules
- CRITICAL: offline node, unexpected stopped guest, root disk at least 90%.
- WARNING: root disk at least 75%, node memory at least 90%.
- No alert for expected stopped guests or QEMU memory accounting alone.

## Validation
Healthy data returned zero alerts and the False branch. A duplicate test workflow simulated HomeProx2 offline, produced one CRITICAL alert, and reached the True branch. Simulation code was removed afterward. AI node-name attribution was also corrected and retested.

## Pending
Publish the original workflow, verify live production notifications, enable the hourly schedule, and evaluate trend monitoring and alert deduplication. The workflow remains paused.

## Skills
Proxmox, Linux, Docker, API integration, n8n, JavaScript, JSON Schema, AI-assisted monitoring, deterministic alerts, and controlled fault simulation.
