# Custom Three-Tier Proxmox Micro-PC Rack

**Status:** Completed — October 19, 2026  
**Category:** Physical Infrastructure / Hardware Design / Systems Integration

## Overview

Designed, iteratively prototyped, 3D printed, and assembled a freestanding three-tier holder for the homelab's three Proxmox VE micro-PC nodes. The holder sits on a rack shelf and organizes the systems horizontally with front I/O accessible and rear cabling routed toward the back.

## Hardware

| Tier | Node | Computer |
|---|---|---|
| Top | HomeProx3 | HP Elite Mini 600 G9 |
| Middle | HomeProx2 | HP Elite Mini 600 G9 |
| Bottom | HomeProx | Dell OptiPlex 3000 (SMBIOS-reported; chassis variant pending confirmation) |

- **CAD:** OpenSCAD; printable STL parts
- **3D printer:** Creality Ender-3 V3 KE
- **Assembly:** M4 bolts and brass heat-set inserts
- **Mounting:** Freestanding holder on a VEVOR 1U rack shelf
- **Design:** Open front and rear, fixed horizontal supports

## HomeProx hardware verification (October 30, 2026)

- System SMBIOS: Dell Inc. / OptiPlex 3000 (supersedes the earlier 7020 Micro identification; form factor not independently confirmed)
- CPU: Intel Core i5-12500T, 6 physical cores / 12 threads
- RAM: 2 × 16 GB DDR4 at 3200 MT/s (approximately 31 GiB usable reported by Linux)
- Disk: Samsung SSD 970 EVO Plus 1 TB NVMe (931.5 GiB reported by `lsblk`)
- Network: Realtek RTL8111/8168-family PCIe Gigabit Ethernet

## Objectives

- Consolidate three compact Proxmox nodes into a maintainable rack arrangement.
- Preserve access to front ports and rear power/network connections.
- Design components suitable for the printer's build volume and available M4 hardware.
- Create a secure, repeatable assembly without relying on adhesive joints.

## Design and Implementation

1. Measured the Dell and HP chassis and modeled a three-tier open-frame holder in OpenSCAD.
2. Iterated joint and mounting geometry through test prints to improve fit and reduce movement.
3. Designed printable structural members and corner posts for a bolted assembly.
4. Revised corner-post mounting to accommodate M4 heat-set inserts, using a previously validated 5.4 mm pilot-hole diameter for the inserts on this printer.
5. Printed the final structural components and assembled the holder.
6. Installed the three micro PCs in the completed rack arrangement and finished cable management and physical checks.

## Troubleshooting and Lessons Learned

- **Joint tolerances:** Early tongue/socket and dovetail prototypes exposed tradeoffs between tight assembly and unwanted play; bolted retention provided a more secure final connection.
- **Hardware compatibility:** Shelf installation required compatible 12-24 rack screws and washers rather than the supplied oversized fasteners.
- **Print constraints:** The design was divided into printable parts and adjusted for reliable FDM fabrication.
- **Serviceability:** Open front/rear access and removable mechanical fasteners simplify maintenance and node replacement.

## Completion and Validation

- [x] CAD design and revisions
- [x] Structural parts printed
- [x] M4 fastening and heat-set insert implementation
- [x] Chassis assembled
- [x] All three nodes installed
- [x] Cable management and final physical checks

## Skills Demonstrated

- Physical infrastructure planning and rack integration
- Parametric CAD and additive manufacturing
- Dimensional measurement and fit testing
- Iterative troubleshooting and prototyping
- Mechanical assembly and serviceability planning
- Virtualization host hardware organization

## Future Documentation

Photographs of the completed holder, sanitized CAD source, and build diagrams may be added separately.
