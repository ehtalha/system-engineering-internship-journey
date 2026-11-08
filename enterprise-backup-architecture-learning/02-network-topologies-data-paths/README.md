# Module 02: Network Topologies & Data Paths (LAN vs. SAN vs. HotAdd)

> **Core principle:** the backup *software* is the same everywhere. What changes performance and production impact is **which path the data takes** from the source to the backup storage.

---

## Table of Contents

1. [Overview](#overview)
2. [Learning Objectives](#learning-objectives)
3. [Path 1: LAN-Based Backup (NBD)](#path-1-lan-based-backup-nbd)
4. [Path 2: SAN-Based Backup (LAN-Free)](#path-2-san-based-backup-lan-free)
5. [Path 3: HotAdd (Appliance / Proxy VM)](#path-3-hotadd-appliance--proxy-vm)
6. [Side-by-Side: Traditional LAN vs. LAN-Free](#side-by-side-traditional-lan-vs-lan-free)
7. [The Three VMware Transport Modes Compared](#the-three-vmware-transport-modes-compared)
8. [Comparison Table](#comparison-table)
9. [How to Choose](#how-to-choose)
10. [Common Pitfalls](#common-pitfalls)
11. [Whiteboard Checklist](#whiteboard-checklist)
12. [Review Questions](#review-questions)
13. [Navigation](#navigation)

---

## Overview

In Module 01 you learned the three tiers (Master, Media Server, Client) and the split between control plane and data plane. This module zooms into the **data plane** and asks one question:

*How does the payload physically travel from the production data to the backup storage?*

There are three main answers when backing up virtual machines (VMware is used as the reference platform):

- **LAN (NBD):** the data travels over the Ethernet network.
- **SAN (LAN-free):** the data travels over the storage network (Fibre Channel or iSCSI), bypassing the LAN.
- **HotAdd:** the VM's disks are attached to a proxy VM inside the hypervisor, so reading needs no network at all.

## Learning Objectives

By the end of this module you will be able to:

- Explain LAN (NBD), SAN (LAN-free), and HotAdd transport modes
- Draw how each data path looks on a whiteboard
- Identify what each mode requires and where it cannot be used
- Choose the right mode for a given environment

---

## Path 1: LAN-Based Backup (NBD)

**NBD** stands for *Network Block Device*. It is the default and simplest transport mode.

- A backup proxy / media server requests the VM disk data from the ESXi host.
- The data travels over the **ESXi management network** (VMkernel interface, typically TCP port 902) to the proxy.
- Works everywhere and needs no special storage access.
- **Downsides:** slowest option, consumes ESXi host CPU and network bandwidth, and can compete with production traffic if the management network is shared.
- NBDSSL is the encrypted variant, with extra CPU cost.

The same idea applies to **traditional agent-based backups** of physical servers: the client sends data to the media server over the LAN.

## Path 2: SAN-Based Backup (LAN-Free)

In SAN transport mode, the backup proxy has **direct, read-only access to the same storage LUNs** that hold the VM disks, using Fibre Channel or iSCSI.

- The proxy reads the VM's blocks directly from the storage array over the SAN.
- The ESXi host and the production LAN are **bypassed** for the read.
- **Upsides:** high throughput and minimal impact on production hosts and LAN.
- **Requirements:** block storage (FC or iSCSI VMFS datastores), SAN zoning/LUN masking so the proxy can see the LUNs, and usually a **physical proxy** (or a server with SAN connectivity).
- **Cannot be used with:** vSAN or NFS datastores, because there is no LUN to present to the proxy.

> **Two meanings of "LAN-free":** in VMware, SAN transport mode is the VM-level version. In classic backup architecture, LAN-free means a client (for example a database server) writes its backup **directly to shared storage or a tape library over Fibre Channel**, and only metadata crosses the LAN.

## Path 3: HotAdd (Appliance / Proxy VM)

In HotAdd mode, the backup proxy runs as a **virtual machine on the same ESXi host or cluster** as the VM being protected.

- A snapshot of the source VM is taken.
- The snapshot's virtual disks are **hot-attached to the proxy VM** as extra virtual SCSI disks.
- The proxy reads them locally through the hypervisor's storage stack, so **no network is used for the read**.
- **Upsides:** good performance without needing a physical SAN-connected proxy, and it is the usual choice for **vSAN and NFS** datastores. It scales by adding proxy VMs.
- **Requirements:** the proxy VM must be able to access the datastore holding the source VM's disks.
- Many backup appliances and virtual proxy deployments (for example virtual proxies or virtual appliances from the major vendors) use HotAdd.

> **Important:** all three modes describe how the proxy **reads** the source data. The path from the proxy **to the backup target** is a separate data path. It usually runs over a dedicated backup network or VLAN (or over FC), and it is often reduced by deduplication (see Module 03).

---

## Side-by-Side: Traditional LAN vs. LAN-Free

```
 A) TRADITIONAL LAN BACKUP              B) LAN-FREE (SAN) BACKUP
 ─────────────────────────              ─────────────────────────

 ┌──────────────────┐                   ┌──────────────────┐
 │  MASTER SERVER   │                   │  MASTER SERVER   │
 └────────┬─────────┘                   └────────┬─────────┘
          ┊ control                               ┊ control
 ┌────────▼─────────┐                   ┌────────▼─────────┐
 │  PRODUCTION      │                   │  PRODUCTION      │
 │  CLIENT / HOST   │                   │  CLIENT / HOST   │
 └────────┬─────────┘                   └──────────────────┘
          ║ DATA over the                  (backup payload does
          ║ production LAN                  NOT cross the LAN)
 ┌────────▼─────────┐                   ┌──────────────────┐
 │  MEDIA SERVER    │                   │  MEDIA SERVER /  │
 └────────┬─────────┘                   │  SAN PROXY       │
          ║ DATA                        └────────┬─────────┘
          ▼                                      ║ FC / iSCSI
 ┌──────────────────┐                   ┌────────▼─────────┐
 │  STORAGE TARGET  │                   │   SAN SWITCH     │
 └──────────────────┘                   └──┬────────────┬──┘
                                           ║ read       ║ write
                                    ┌──────▼─────┐ ┌────▼───────┐
                                    │ PRODUCTION │ │  STORAGE   │
                                    │ LUNs       │ │  TARGET    │
                                    └────────────┘ └────────────┘

 Legend:  ┊ = control plane (thin)    ║ = data plane (thick)
```

**Key difference:** in (A) the backup payload competes with production traffic on the LAN. In (B) the payload moves over the storage network, and only small control messages use the LAN.

## The Three VMware Transport Modes Compared

All three start the same way: the master instructs the backup software, which asks vCenter to take a **snapshot** of the VM (control plane). Then the read path differs.

```
 MODE 1: NBD (LAN)
 ┌────────────┐   ┌────────────┐   mgmt network    ┌──────────┐   ┌────────┐
 │ VM disk on │──▶│ ESXi host  │═══ (TCP 902) ════▶│  Backup  │══▶│ Target │
 │ datastore  │   │ reads+sends│                   │  Proxy   │   └────────┘
 └────────────┘   └────────────┘                   └──────────┘
                   ESXi CPU + NIC used; LAN used

 MODE 2: SAN (LAN-FREE)
 ┌────────────┐          FC / iSCSI (direct, read-only)  ┌──────────┐   ┌────────┐
 │ VM disk on │═════════════════════════════════════════▶│ Physical │══▶│ Target │
 │ SAN LUN    │                                          │  Proxy   │   └────────┘
 └────────────┘                                          └──────────┘
   ESXi host and LAN are BYPASSED for the read

 MODE 3: HotAdd
 ┌──────────────── same ESXi host / cluster ────────────────┐
 │ ┌────────────┐  attach snapshot disk   ┌──────────────┐  │   ┌────────┐
 │ │ VM disk on │────(virtual SCSI)──────▶│  Proxy VM    │══╪══▶│ Target │
 │ │ datastore  │                         │  (HotAdd)    │  │   └────────┘
 │ └────────────┘                         └──────────────┘  │
 └──────────────────────────────────────────────────────────┘
   Read happens inside the host; no network used for the read
```

## Comparison Table

| Feature | NBD (LAN) | SAN (LAN-free) | HotAdd |
|---------|-----------|----------------|--------|
| Read path | ESXi host over management network | Direct from SAN (FC/iSCSI) | Virtual SCSI inside the host |
| Relative speed | Slowest | Fastest (typically) | Fast |
| Impact on ESXi host | High (CPU and NIC) | Lowest | Moderate (storage I/O) |
| Impact on production LAN | High | Minimal | Minimal for the read |
| Proxy type | Physical or virtual | Usually physical | Virtual (proxy VM) |
| Needs SAN access for proxy | No | Yes (zoning / LUN masking) | No |
| Works with vSAN / NFS | Yes | No | Yes |
| Setup effort | Lowest | Highest | Medium |

## How to Choose

- **Block storage (FC/iSCSI), large VMs, maximum speed, minimal host impact:** SAN transport with a physical proxy.
- **vSAN, NFS, or no SAN access for the proxy:** HotAdd with proxy VMs (add more proxies to scale).
- **Small environment, quick start, or fallback:** NBD.
- Most products try modes in a configured order (for example SAN, then HotAdd, then NBD) and **fall back automatically**, so verify which mode a job actually used.

## Common Pitfalls

- **SAN mode:** the proxy must have **read-only** access to the datastore LUNs. If a Windows proxy is allowed to mount and write to VMFS volumes, it can damage them, so disable automount and bring disks online carefully.
- **NBD:** sharing the ESXi management network with backup traffic can hurt management and production traffic. Use a dedicated VMkernel / backup VLAN where possible.
- **HotAdd:** the proxy VM must be able to reach the datastore of the VM being backed up. A proxy on a host without access cannot use HotAdd for that VM.
- **Silent fallback:** a job may quietly fall back to NBD and run slowly. Check the job details for the transport mode used.
- **Snapshots:** very large or very busy VMs can suffer from snapshot stun and long snapshot consolidation. Plan backup windows accordingly.
- **Incrementals:** use Changed Block Tracking (CBT) so only changed blocks are read, regardless of mode.

---

## Whiteboard Checklist

- [ ] Draw the source (VM disk / datastore) on the left and the backup target on the right
- [ ] Draw the proxy / media server in the middle
- [ ] For NBD: route the data through the ESXi host and the LAN
- [ ] For SAN: draw the FC/iSCSI link from the LUN straight to the proxy, bypassing the host and LAN
- [ ] For HotAdd: draw the proxy VM inside the same ESXi host with the disk attached
- [ ] Draw the separate proxy-to-target path (dedicated backup network or FC)
- [ ] Say clearly: **"The mode describes how the data is read, not how it is stored."**

---

## Review Questions

1. A company runs VMware on **vSAN** and wants the fastest possible backups with minimal impact on the production LAN. Which transport mode would you choose and why? Explain why SAN mode is not available.
2. Your backup jobs for VMs on a Fibre Channel SAN are running much slower than expected, and the job logs show the **NBD** mode was used. What does this tell you, and what would you check?

## Answers

1. Which transport mode suits VMware on vSAN with minimal production LAN impact, and why is SAN mode unavailable?

HotAdd is the appropriate choice.

SAN mode requires the backup proxy to access a physical storage LUN directly. vSAN pools the local storage of the ESXi hosts and exposes no external LUN, so SAN mode cannot be used.
In HotAdd mode, the snapshot disks of the source VM are attached to a proxy VM as virtual SCSI disks. The proxy reads them inside the host through the hypervisor storage stack, so no network is used for the read and the production LAN is not affected.
Deploy a proxy VM on each host or cluster and add more proxies to scale.
Send the proxy-to-target traffic over a dedicated backup network or VLAN, and use deduplication to reduce the volume written.

2. Backup jobs for VMs on a Fibre Channel SAN run slowly and the logs show NBD mode. What does this indicate and what should be checked?
NBD mode means the data is being read through the ESXi host over the network, which is slower and loads both the host and the network. On an FC SAN, SAN mode should normally be used, so this indicates that SAN mode failed and the job silently fell back to NBD.
---
Items to check:
 1.	The proxy can see the datastore LUNs (FC zoning and array LUN masking)
 2. The proxy is a physical (or SAN-connected) server, as virtual proxies generally cannot use SAN mode
 3. The datastore type supports SAN mode (VMFS on block storage; not NFS, vSAN, or VVol in some products)
 4. The transport mode order or a forced NBD setting in the job or proxy configuration
 5. The job log for the SAN failure message that preceded the fallback
 6.	DNS resolution between the backup server, ESXi hosts, and vCenter

## Navigation

- Previous: [Module 01: Three-Tier Backup Topology](../01-three-tier-backup-topology/)
- Next: [Module 03: Storage Targets & Deduplication](../03-storage-targets-deduplication/)
- Back to the [main README](../README.md)
