# Cisco UCS C240 M6S Client Site Deployment: Rack Mounting, RAID, Certificate and OS Installation

| Field | Details |
|---|---|
| **Document Type** | Task Documentation (Client Site Deployment Phase) |
| **Task Date** | 09 November 2026 |
| **Location** | Client Data Center |
| **Equipment** | 2 x Cisco UCS C240 M6S Rack Servers |
| **Performed By** | Senior engineers of the System Team (execution); Emdadul Haque Talha, Intern Engineer (observation and assistance) |
| **Related Document** | Staging phase documentation (08 November 2026, company data center) |
| **Status** | Completed; client verification performed; UAT evidence collected |

---

## Table of Contents

1. [Overview](#1-overview)
2. [Scope](#2-scope)
3. [Key Concepts](#3-key-concepts)
4. [Deployment Procedure](#4-deployment-procedure)
5. [Troubleshooting Record: RAID Not Detected](#5-troubleshooting-record-raid-not-detected)
6. [RAID Configuration](#6-raid-configuration)
7. [Certificate Configuration](#7-certificate-configuration)
8. [Operating System Installation](#8-operating-system-installation)
9. [Network Configuration](#9-network-configuration)
10. [Client Verification and UAT](#10-client-verification-and-uat)
11. [Configuration Summary](#11-configuration-summary)
12. [Lessons Learned](#12-lessons-learned)
13. [Quick Checklist](#13-quick-checklist)

---

## 1. Overview

This document records the second phase of the Cisco UCS C240 M6S deployment for the client. In the first phase (08 November 2026), the servers were staged in the company data center. In this phase (09 November 2026), the servers were installed at the client data center, configured, and verified by the client.

The same procedure was applied to both servers. The tasks were performed mainly by the senior engineers, while the intern observed and assisted.

## 2. Scope

**Included in this document:**

- Rack mounting and cabling at the client data center
- CIMC access and log cleanup
- Troubleshooting of the RAID detection issue
- RAID 5 and hot spare configuration
- Manual certificate installation
- Windows Server installation using a bootable USB drive
- Management and CIMC IP configuration
- Client verification and UAT evidence collection

**Not included:**

- Hardware staging (see the staging phase documentation)
- Application, role or service configuration beyond the operating system installation

## 3. Key Concepts

| Term | Explanation |
|---|---|
| **CIMC** | Cisco Integrated Management Controller. An out-of-band management controller used to monitor and manage the server hardware remotely. |
| **MGT Port** | Port used for management of the server's operating system. It was assigned a client-allocated IP address. |
| **MZ Port** | Militarized Zone (MZ) network port, connected to the corresponding network segment of the client. No IP address was required on the server for this port. |
| **vKVM** | Virtual KVM console available through CIMC. It provides remote access to the server console and power controls. |
| **Power Cycle (Recycle)** | Powering the server off and on again. This restarts the host server itself, unlike a CIMC reboot, which restarts only the management controller. |
| **RAID 5** | Storage configuration that stripes data across drives with distributed parity. It tolerates the failure of one drive. Usable capacity is that of (N-1) drives. |
| **Dedicated Hot Spare** | A drive assigned to a RAID group that stays idle until a member drive fails, after which it automatically takes over and the array is rebuilt. |
| **Virtual Drive** | A logical disk created from one or more physical drives by the RAID controller. |
| **Security Management (CIMC)** | CIMC section used to manage security settings, including certificates for the web interface. |
| **UAT** | User Acceptance Testing. The client's formal verification that the delivered system meets the agreed requirements. |

## 4. Deployment Procedure

### Step 1: Rack Mounting

1. After the pre-entry verification process, the team entered the client data center.
2. Both servers were mounted in the rack with the assistance of the client-side technicians.

### Step 2: Cabling

The technicians connected cables from the network switch to each server. Three cables were connected at the rear of each server:

| No. | Port | Purpose |
|---|---|---|
| 1 | **MGT** | Operating system (OS) management port; a client-allocated IP address was assigned |
| 2 | **MZ** | Militarized Zone (MZ) network port; no IP address was required |
| 3 | **IMC (CIMC)** | Out-of-band hardware management (CIMC); a client-allocated IP address was assigned |

### Step 3: Power On

The servers were powered on after cabling was completed.

### Step 4: Access CIMC

1. Connected a laptop to the server's CIMC port using a **UTP (Ethernet) cable**.
2. Opened the previously configured CIMC IP address in a web browser.
3. Logged in as **administrator**.
4. Cleared all **previous logs** (those generated during staging and transport).

## 5. Troubleshooting Record: RAID Not Detected

| Item | Details |
|---|---|
| **Affected Servers** | Both servers |
| **Symptom** | The RAID controller was not displayed in the CIMC Storage section. |
| **Action 1** | Rebooted. The RAID controller was still not detected. |
| **Action 2** | Performed a **power recycle** several times, first from the CIMC **Host Power** option and then from the **vKVM** console. |
| **Result** | The RAID controller appeared in the Storage section. |

**Observation:** This is the same behavior as seen on the second server during staging, where a power recycle resolved the issue. In this deployment, repeated power recycles were required on both servers.

## 6. RAID Configuration

After the RAID controller became visible, the senior engineer configured the storage using the RAID controller options in CIMC.

| Parameter | Configuration |
|---|---|
| **RAID Level** | RAID 5 |
| **Member Drives** | Drives 1, 2, 3, 4, 5 and 6 (6 x 3.84 TB SAS SSD) |
| **Hot Spare** | Drive 7, assigned as a **dedicated hot spare** |
| **Approximate Usable Capacity** | 19.2 TB (5 x 3.84 TB, calculated; actual value may differ slightly due to formatting) |

**Purpose of the hot spare:** If any of the six RAID 5 drives fails, drive 7 automatically takes its place and the array is rebuilt, reducing the time the array runs in a degraded state.

## 7. Certificate Configuration

After the RAID configuration, the senior engineer installed a certificate **manually** from the **Security Management** section of the CIMC web interface.

## 8. Operating System Installation

**Operating System:** Windows Server Standard (Desktop Experience)

**Method:** Bootable USB drive (pendrive)

### Procedure

1. Plugged the bootable USB drive into a **USB port at the rear of the server**.
2. From the **vKVM** console, powered the system **off**.
3. Powered the system **on**.
4. Selected the USB drive as the boot device.
5. Selected **Windows Server Standard (Desktop Experience)** in the installer.
6. At the disk partitioning step, the senior engineer confirmed the OS partition size with the client. The client approved **200 GB** for the operating system.
7. Created a new partition of 200 GB (the installer accepts the size in **MB**, so the value was entered in MB; 200 GB equals 204,800 MB at 1 GB = 1,024 MB) and selected it as the installation target.
8. Completed the installation.

## 9. Network Configuration

After the OS installation, the senior engineer assigned IP addresses based on the allocation provided by the client.

| Port | IP Configuration |
|---|---|
| **CIMC port** | Client-allocated IP address assigned (hardware management) |
| **MGT port** | Client-allocated IP address assigned (OS management) |
| **MZ port** | No IP address required (Militarized Zone connection) |

## 10. Client Verification and UAT

1. The senior engineer informed the client that the configuration was complete and asked them to verify it.
2. The client applied their own **firewall policies** so that they could test connectivity and confirm that everything was working.
3. After the client confirmed, the senior engineer took screenshots as evidence for the official **UAT (User Acceptance Test)** document.

## 11. Configuration Summary

| Item | Server 1 | Server 2 |
|---|---|---|
| Rack mounting and cabling | Done | Done |
| CIMC access and log cleanup | Done | Done |
| RAID controller detected | After power recycles | After power recycles |
| RAID 5 (drives 1 to 6) | Configured | Configured |
| Dedicated hot spare (drive 7) | Configured | Configured |
| Certificate (manual) | Installed | Installed |
| OS | Windows Server Standard (Desktop Experience) | Windows Server Standard (Desktop Experience) |
| OS partition | 200 GB | 200 GB |
| CIMC and MGT IP | Client-allocated | Client-allocated |
| MZ port IP | Not required | Not required |
| UAT screenshots | Collected | Collected |

The procedure was identical for both servers.

## 12. Lessons Learned

1. **Power cycle versus CIMC reboot.** When the RAID controller is not detected, a CIMC reboot may not help. A host power cycle (from the Host Power option or the vKVM console) may be needed, sometimes more than once.
2. **Detect the RAID controller before configuring storage.** The RAID configuration and OS installation depend on the controller being visible.
3. **Use a hot spare.** Dedicating drive 7 as a hot spare adds protection to the RAID 5 array at the cost of one drive of capacity.
4. **Confirm sizing with the client.** The OS partition size (200 GB) was confirmed with the client before installation.
5. **Collect evidence for UAT.** Screenshots taken after client verification support the acceptance document.
6. **Know the partition unit.** The Windows installer takes partition sizes in MB, so convert GB values carefully.

## 13. Quick Checklist

- [ ] Mount servers in the rack
- [ ] Technicians connect MGT, MZ and IMC (CIMC) cables to the switch
- [ ] Power on the servers
- [ ] Connect the laptop to the CIMC port; log in as administrator
- [ ] Clear previous logs
- [ ] Verify the RAID controller is visible (if not: reboot, then power cycle from Host Power or vKVM)
- [ ] Create RAID 5 with drives 1 to 6; assign drive 7 as dedicated hot spare
- [ ] Install the certificate manually in Security Management
- [ ] Install Windows Server Standard (Desktop Experience) from the bootable USB; 200 GB OS partition
- [ ] Assign client IPs to the CIMC and MGT ports (none for MZ)
- [ ] Client verifies and applies firewall policies
- [ ] Capture screenshots for the UAT document

---

*Prepared by: Emdadul Haque Talha, Intern Engineer, System Team, Express Systems Limited*
