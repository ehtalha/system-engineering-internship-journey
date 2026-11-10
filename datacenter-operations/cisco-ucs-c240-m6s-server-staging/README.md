# Cisco UCS C240 M6S Server Staging: Hardware Installation and CIMC Configuration

| Field | Details |
|---|---|
| **Document Type** | Task Documentation (Staging Phase) |
| **Task Date** | 08 November 2026 |
| **Location** | Company Data Center (Express Systems Limited) |
| **Client** | NRB Bank |
| **Equipment** | 2 x Cisco UCS C240 M6S Rack Servers |
| **Performed By** | Emdadul Haque Talha (Intern Engineer) with two senior engineers of the System Team |
| **Status** | Staging phase completed; servers handed over to store for client delivery |

---

## Table of Contents

1. [Overview](#1-overview)
2. [Scope](#2-scope)
3. [Hardware Specification Summary](#3-hardware-specification-summary)
4. [Key Concepts](#4-key-concepts)
5. [Staging Procedure](#5-staging-procedure)
6. [CIMC Configuration](#6-cimc-configuration)
7. [Verification](#7-verification)
8. [Troubleshooting Record](#8-troubleshooting-record)
9. [Final Handover](#9-final-handover)
10. [Remaining Work (Client Site Phase)](#10-remaining-work-client-site-phase)
11. [Lessons Learned](#11-lessons-learned)
12. [Quick Checklist](#12-quick-checklist)

---

## 1. Overview

Our company procured two **Cisco UCS C240 M6S** rack servers for the client, NRB Bank. Before delivery, the servers were prepared (staged) in the company data center according to the client's hardware requirements. Staging consisted of installing storage drives, adjusting memory, installing network interface cards (NICs), racking the servers, configuring the out-of-band management controller (CIMC), and verifying that all installed components were detected correctly.

This document records the full procedure so that it can be understood and reproduced in future tasks.

## 2. Scope

**Included in this document (Phase 1: Staging at the company data center):**

- Physical installation of SAS SSD drives
- Memory (RAM) adjustment according to the client requirement
- Installation of two NIC cards in PCIe slots
- Racking and power connection
- Initial CIMC network configuration
- Hardware verification through the CIMC web interface
- Troubleshooting of the RAID detection issue on the second server

**Not included (Phase 2: Client site):**

- Operating system installation
- Final network and system configuration
- Any application or service setup

## 3. Hardware Specification Summary

| Component | Details (per server) |
|---|---|
| Server Model | Cisco UCS C240 M6S |
| Storage Drives | 7 x 3.84 TB SAS SSD |
| Raw Storage Capacity | 26.88 TB (7 x 3.84 TB, calculated) |
| Memory as received | 16 x 64 GB DIMMs (1024 GB) |
| Memory after adjustment | 8 x 64 GB DIMMs (512 GB, calculated); 8 DIMMs removed per client requirement |
| Network Cards | 2 x NIC cards installed in PCIe slots |
| Management Interface | CIMC (Cisco Integrated Management Controller) |

## 4. Key Concepts

| Term | Explanation |
|---|---|
| **CIMC** | Cisco Integrated Management Controller. A dedicated management processor that allows remote monitoring and configuration of the server (hardware inventory, power control, storage, logs) independent of the operating system. |
| **Dummy (blank) drive** | A filler placed in an empty drive bay to maintain proper airflow and cooling. It must be removed before inserting a real drive. |
| **DIMM / RAM stick** | A memory module installed in a memory slot on the motherboard. Slot numbering matters for correct population. |
| **PCIe slot** | Expansion slot used to install add-on cards such as NICs. Unused rear slots are covered by blanking plates (dummy covers). |
| **NIC** | Network Interface Card, providing network connectivity to the server. |
| **RAID** | Redundant Array of Independent Disks. A storage technique combining multiple drives for performance, capacity or redundancy. The RAID controller must be visible in CIMC for the drives to be configured. |
| **PDU** | Power Distribution Unit. Supplies power to equipment installed in the rack. |
| **Dedicated NIC mode** | CIMC network mode in which the management traffic uses the server's dedicated management port only. |

## 5. Staging Procedure

The following steps were performed on each server.

### Step 1: Install Storage Drives

1. Removed the dummy (blank) drive fillers from the front drive bays.
2. Inserted **7 x 3.84 TB SAS SSD** drives into the bays.

### Step 2: Adjust Memory (RAM)

1. Opened the server chassis and removed the top cover to access the memory slots.
2. Found **16 x 64 GB** DIMMs pre-installed.
3. According to the client's required memory capacity, identified **8 DIMMs** for removal by checking the slot numbers.
4. Removed those 8 DIMMs, leaving 8 x 64 GB installed.

### Step 3: Install NIC Cards

1. At the rear of the server, opened one PCIe riser/slot area.
2. Removed **two dummy (blank) covers**.
3. Inserted **two NIC cards** and seated them in the PCIe slots.

### Step 4: Close Chassis and Rack the Server

1. Verified that all installed components (drives, DIMMs, NICs) were seated correctly.
2. Closed the chassis and top cover.
3. Mounted the server in the data center rack.

### Step 5: Power Connection and Power On

1. Connected the power cables from the rack **PDU** to the server.
2. After power was supplied, switched on the server using the **front power button**.
3. Connected a **monitor and keyboard** at the rear of the server to access the console for CIMC configuration.

## 6. CIMC Configuration

During boot, the senior engineer entered the CIMC configuration screen and applied the following settings:

| Parameter | Value |
|---|---|
| IP Version | IPv4 |
| IP Address / Subnet Mask | Assigned as per plan|
| NIC Mode | **Dedicated** |
| NIC Redundancy | **None** |

After saving the settings, a laptop was connected to the server's management port using a **UTP (Ethernet) cable**. The CIMC web interface was opened in a browser using the configured IP address, and the engineer logged in as **administrator**.

## 7. Verification

From the CIMC web interface, the following items were checked:

- All **7 SAS SSD drives** are detected and listed.
- The **RAID controller** and storage configuration are visible in the Storage section.
- Installed **memory** is detected correctly.
- Installed **NIC cards** are detected.

Screenshots of each verified component were taken as evidence for both servers.

## 8. Troubleshooting Record

### 8.1 Server 1: Components Not Initially Displayed

| Item | Details |
|---|---|
| **Symptom** | Some components were not showing properly in CIMC. |
| **Action** | Rebooted CIMC. |
| **Result** | All components appeared correctly. Screenshots were taken. |

### 8.2 Server 2: RAID Not Detected in Storage Section

| Item | Details |
|---|---|
| **Symptom** | The RAID controller was not shown in the CIMC Storage section. |
| **Actions attempted** | 1. Rebooted the server/CIMC multiple times. <br> 2. Powered down the server, opened the chassis and inspected the internal connections, then powered up again. <br> 3. A second senior engineer investigated further. |
| **Resolution** | The second senior engineer performed a **power recycle** of the server. |
| **Result** | The RAID controller was detected. Screenshots were taken. |

**Observation:** A simple CIMC reboot did not resolve the issue on Server 2, whereas a full power recycle did. This suggests the controller needed a complete power reset to be re-initialized.
#### Power Recycle Procedure

The issue on Server 2 was resolved by performing a **power recycle** of the server. A power recycle can be initiated from the CIMC web interface in either of the following ways:

1. **Host Power menu (CIMC):** The *Host Power* menu in CIMC provides a **Power Cycle** option, which powers the server off and then back on.
2. **vKVM (Virtual KVM Console):** The vKVM console also provides a power control menu from which the server can be power cycled.

After the power recycle was completed, the RAID controller and the installed drives were detected correctly in the Storage section of CIMC, and the verification screenshots were captured.

**Note:** A power recycle differs from a CIMC reboot. A CIMC reboot restarts only the management controller, whereas a power recycle restarts the host server itself. This may explain why the CIMC reboot did not resolve the issue on Server 2.
## 9. Final Handover

1. Both servers were powered down.
2. All work was completed in the company data center.
3. The servers were handed over to the company **store**, from where they will be dispatched to the client site.

## 10. Remaining Work (Client Site Phase)

Once the servers reach the client site, the team will perform:

- Rack mounting and cabling at the client data center
- Final network configuration
- Operating system installation
- Remaining setup and configuration as per client requirements

## 11. Lessons Learned

1. **Verify hardware through CIMC after installation.** CIMC provides a quick way to confirm drives, RAID, memory and NICs are detected before shipping.
2. **Try a CIMC reboot first, then escalate.** For detection problems: CIMC reboot, then reseat/inspect components, then full power recycle.
3. **Identical servers can behave differently.** Server 1 and Server 2 had the same configuration but needed different fixes.
4. **Keep evidence.** Screenshots of the verified inventory serve as proof of correct staging.
5. **Follow slot numbering when removing or installing DIMMs** to keep the memory population correct.
6. **Remove dummy drives and blanking plates carefully** and keep them for reuse if required.

## 12. Quick Checklist

- [ ] Remove dummy drive fillers
- [ ] Install 7 x 3.84 TB SAS SSD
- [ ] Open chassis; remove 8 of 16 DIMMs per requirement (check slot numbers)
- [ ] Remove 2 PCIe dummy covers; install 2 NIC cards
- [ ] Close chassis; mount in rack
- [ ] Connect PDU power; power on via front button
- [ ] Connect monitor and keyboard; configure CIMC (IPv4, Dedicated NIC, Redundancy: None)
- [ ] Connect laptop via UTP; log in to CIMC web interface
- [ ] Verify drives, RAID, memory and NICs; capture screenshots
- [ ] If a component is missing: CIMC reboot, then inspect connections, then power recycle
- [ ] Power down; hand over to store

---

*Prepared by: Emdadul Haque Talha, Intern Engineer, System Team, Express Systems Limited*
