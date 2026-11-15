# Module 06: Visual Architecture Challenge

> **Goal:** apply everything from Modules 01 to 05 to realistic design problems. For each scenario, produce a layout, justify every decision with numbers and constraints, and name the main risk.

---

## Table of Contents

1. [Overview](#overview)
2. [How This Module Works](#how-this-module-works)
3. [Design Framework](#design-framework)
4. [Answer Template](#answer-template)
5. [Useful Numbers](#useful-numbers)
6. [Scenario 1: 50 TB SQL Server on VMware with No Production LAN Impact](#scenario-1-50-tb-sql-server-on-vmware-with-no-production-lan-impact)
7. [Scenario 2: Twenty Branch Offices over Slow WAN Links](#scenario-2-twenty-branch-offices-over-slow-wan-links)
8. [Scenario 3: Ransomware-Resilient Design for a Financial Institution](#scenario-3-ransomware-resilient-design-for-a-financial-institution)
9. [Evaluation Rubric](#evaluation-rubric)
10. [Navigation](#navigation)

---

## Overview

Modules 01 to 05 covered the architecture of enterprise backup piece by piece:

| Module | Concept used in design |
|--------|------------------------|
| 01 | Master / media / client tiers, control plane vs. data plane, catalog |
| 02 | LAN (NBD), SAN (LAN-free), and HotAdd data paths |
| 03 | Source-side vs. target-side deduplication, storage tiers |
| 04 | Immutability, air gaps, zero trust, clean-room recovery |
| 05 | Tiered (NetBackup-style) vs. scale-out (Rubrik/Cohesity-style) architectures |

This module combines them. Each scenario gives requirements and constraints, and the task is to design the architecture.

## How This Module Works

1. Read a scenario and write down the numbers (data size, change rate, window, links).
2. Produce a design: components, data path, tiers, security, and recovery.
3. Draw the layout as an ASCII diagram or describe it in words.
4. Critique your own design against the rubric, then compare with the reference solution.

---

## Design Framework

Answer these seven questions for every scenario:

| # | Question | Related module |
|---|----------|----------------|
| 1 | **Workload:** what is being protected, how big, how fast does it change, and what consistency does it need? | 01 |
| 2 | **Data path:** how is data read from the source (LAN, SAN, HotAdd, agent)? | 02 |
| 3 | **Brain and movers:** where do the control plane, catalog, and media servers (or cluster nodes) live? | 01, 05 |
| 4 | **Deduplication and tiers:** where is data deduplicated (source or target) and which storage tiers are used? | 03 |
| 5 | **Network and bandwidth:** does the design meet the backup window, and is backup traffic isolated from production? | 02 |
| 6 | **Security:** how are immutability, identity separation, and air gaps handled? | 04 |
| 7 | **Recovery:** can the design meet the RTO and RPO, and how is recovery tested? | 04 |

## Answer Template

```
   SOURCE ──(data path?)──▶ DATA MOVER ──(which network?)──▶ STORAGE TIER(S)
     │                         │                                   │
     │                         │                                   ▼
     │                         │                         VAULT / OFFLINE COPY
     │                         │
     └────────── CONTROL PLANE: who is the brain, and where is the catalog? ──────┐
                                                                                  ▼
                                                                   (protect it: HA + catalog backup)
```

For each scenario, provide:

- **Component list:** servers, appliances, networks, and tiers
- **Data path:** arrows from source to final copy
- **Key decisions:** each with a reason tied to a requirement
- **Calculation:** at least one bandwidth or window check
- **Main risk:** the weakest point of your design and how you would reduce it

---

## Useful Numbers

Theoretical rates; plan for lower real-world throughput.

| Link | Approximate throughput | Data per hour |
|------|------------------------|---------------|
| 20 Mbps WAN | 2.5 MB/s | about 9 GB |
| 100 Mbps WAN | 12.5 MB/s | about 45 GB |
| 1 Gbps Ethernet | 125 MB/s | about 450 GB |
| 10 Gbps Ethernet | 1.25 GB/s | about 4.5 TB |
| 16 Gb Fibre Channel | about 1.6 GB/s | about 5.8 TB |

Quick formulas:

- Required rate = data to move / backup window
- Daily changed data = total data x daily change rate
- Rough deduplication check: stored data is approximately first full + (daily change x days retained)

---

## Scenario 1: 50 TB SQL Server on VMware with No Production LAN Impact

**Environment**

- VMware vSphere cluster on a Fibre Channel SAN (VMFS datastores)
- One SQL Server virtual machine with about 50 TB of databases across several virtual disks
- Production LAN is 10 GbE and shared with application traffic
- A separate backup network (10 or 25 GbE) can be built, and the FC fabric has free ports

**Requirements**

- Production LAN traffic must **not** be affected by backups
- Backup window: 8 hours nightly
- RPO: 24 hours for the full database, with transaction log backups every 15 minutes
- RTO: 4 hours to recover the database
- Daily change rate: about 3%
- Retention: 35 days on disk and 1 year long term
- Backups must be application-consistent

**Design task**

Design the end-to-end backup architecture. Show how data is read, where it travels, where it is deduplicated and stored, and how the long-term copy is kept. Include a window calculation and state whether a nightly full backup is realistic.

---

## Scenario 2: Twenty Branch Offices over Slow WAN Links

**Environment**

- 20 branch offices, each with about 2 TB of data (a file server and a few virtual machines on one ESXi host)
- Each branch has a 20 Mbps WAN link to the central data center
- No IT staff at the branches
- The central data center has storage capacity and can host a vault

**Requirements**

- Daily change rate: about 2%
- RPO: 24 hours
- Restoring a single file at a branch should take minutes, not hours
- Retention: 90 days
- Central management with minimal on-site effort
- The backup window at each branch is overnight

**Design task**

Design the architecture for all branches and the central site. Show where deduplication happens, how the first full backup is handled, how local restores are served, and how the central copy is protected. Include a WAN calculation.

---

## Scenario 3: Ransomware-Resilient Design for a Financial Institution

**Environment**

- About 200 TB of data: 300 VMware virtual machines and 10 physical Oracle database servers
- An existing tape library
- Current backups sit on a NAS that is joined to the production Active Directory domain
- A second data center is located 40 km away

**Requirements**

- Regulation requires 7-year retention for certain data
- RTO: 8 hours for critical systems; RPO: 24 hours
- The design must survive the compromise of **domain administrator** credentials
- Recovery must be safe even if the most recent backups are infected

**Design task**

Redesign the backup architecture for ransomware resilience. Show the network boundaries, immutable storage, vault and air gaps, identity design, the long-term retention approach, and the recovery process.

---

## Evaluation Rubric

Each design is checked against these points:

| Area | What a strong answer shows |
|------|----------------------------|
| Requirements met | Every stated requirement is addressed explicitly |
| Data path | Correct choice of LAN, SAN, HotAdd, or agent path, with a reason |
| Numbers | A bandwidth or window calculation that supports the design |
| Deduplication and tiers | Correct placement of deduplication and appropriate storage tiers |
| Security | Immutability, separate identity, and air gap where needed |
| Recovery | RTO and RPO are met and recovery is tested |
| Control plane protection | The brain and catalog are protected (HA and catalog backup) |
| Trade-offs and risks | The weakest point is identified and mitigated |

---

## Navigation

- Previous: [Module 05: Vendor Architecture Comparisons](../05-vendor-architecture-comparisons/)
- Back to the [main README](../README.md)
