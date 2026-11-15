# Module 05: Vendor Architecture Comparisons (Whiteboard Breakdown)

> **Core idea:** vendors differ less in features than in **architecture**. Compare any platform by asking three questions: *where is the brain, where does the data land, and how does it scale?*

---

## Table of Contents

1. [Overview](#overview)
2. [Learning Objectives](#learning-objectives)
3. [Two Design Philosophies](#two-design-philosophies)
4. [Architecture A: NetBackup with NetBackup Appliances](#architecture-a-netbackup-with-netbackup-appliances)
5. [Architecture B: Scale-Out Hyperconverged Clusters (Rubrik and Cohesity)](#architecture-b-scale-out-hyperconverged-clusters-rubrik-and-cohesity)
6. [Industry Note: Veritas NetBackup Is Now Part of Cohesity](#industry-note-veritas-netbackup-is-now-part-of-cohesity)
7. [Side-by-Side Comparison](#side-by-side-comparison)
8. [How to Choose](#how-to-choose)
9. [Common Pitfalls](#common-pitfalls)
10. [Whiteboard Checklist](#whiteboard-checklist)
11. [Review Questions](#review-questions)
12. [Answers](#answers)
13. [References](#references)
14. [Navigation](#navigation)

---

## Overview

Modules 01 to 04 described a generic backup architecture. This module maps that generic model onto two real design styles:

- **Tiered (traditional):** separate roles for the management server, media servers, and storage, as used by Veritas / Cohesity NetBackup, here packaged with turnkey NetBackup Appliances.
- **Scale-out hyperconverged:** a cluster of identical nodes in which control, metadata, compute, and storage are combined, as used by Rubrik and Cohesity DataProtect.

> Vendor products change quickly. Treat this module as an architectural comparison and verify product details against current vendor documentation before making design or purchasing decisions.

## Learning Objectives

By the end of this module you will be able to:

- Draw a NetBackup-with-appliances architecture and a scale-out cluster architecture on a whiteboard
- Explain where the control plane, data plane, and catalog live in each
- Compare scaling, deduplication scope, immutability, and management between the two styles
- Explain how the Cohesity acquisition of Veritas changes the NetBackup picture
- Recommend an approach for a given environment and justify it

---

## Two Design Philosophies

```
 A) TRADITIONAL / TIERED                    B) SCALE-OUT / HYPERCONVERGED
 ───────────────────────                    ─────────────────────────────

 ┌──────────────────────┐                   ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
 │ PRIMARY SERVER       │                   │NODE 1│ │NODE 2│ │NODE 3│ │NODE 4│
 │ (brain: policy +     │                   │ ctrl │ │ ctrl │ │ ctrl │ │ ctrl │
 │  catalog)            │                   │ meta │ │ meta │ │ meta │ │ meta │
 └──────────┬───────────┘                   │ media│ │ media│ │ media│ │ media│
            ┊ control                       │ disk │ │ disk │ │ disk │ │ disk │
     ┌──────┴───────┐                       └──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘
     ▼              ▼                       ═══ one distributed file system ═══
 ┌─────────┐  ┌─────────┐                   metadata, dedupe, protection shared
 │ MEDIA 1 │  │ MEDIA 2 │                   by all nodes
 └────┬────┘  └────┬────┘
      ▼            ▼
 ┌───────────────────────┐
 │ STORAGE (disk/cloud/  │
 │ tape)                 │
 └───────────────────────┘

 Scale: add media servers / storage         Scale: add nodes (capacity and
 Brain: central (needs HA)                  performance together)
                                            Brain: distributed (no master node)
```

---

## Architecture A: NetBackup with NetBackup Appliances

NetBackup follows the classic three-tier model from Module 01: a **primary server** (called the *master server* in older versions) holding policies and the catalog, **media servers** that move and store data, and **clients/agents** on the protected systems.

**NetBackup Appliances** package the media server (and often the primary server) with hardware, storage, and a hardened operating environment in a single turnkey system. On the Flex appliance platform, containerized NetBackup services consolidate primary, media, deduplication, and WORM storage on one appliance.

```
   PRODUCTION: VMs / databases / file servers (clients, agents, VM proxies)
                          │ data plane (LAN, SAN, or HotAdd)
                          ▼
 ┌─────────────────────────────────────────────────────────────────────┐
 │ NETBACKUP APPLIANCES (media servers with built-in storage)          │
 │ ┌─────────────────────────┐    ┌─────────────────────────┐          │
 │ │ APPLIANCE 1             │    │ APPLIANCE 2             │  + more  │
 │ │ media service           │    │ media service           │          │
 │ │ dedupe pool (MSDP)      │    │ dedupe pool (MSDP)      │          │
 │ │ immutable (WORM) disk   │    │ immutable (WORM) disk   │          │
 │ └────────────┬────────────┘    └────────────┬────────────┘          │
 └──────────────┼──────────────────────────────┼───────────────────────┘
                ┊ control and catalog updates    ┊
                ▼                               ▼
        ┌──────────────────────────────────────────────┐
        │ PRIMARY SERVER (policies + catalog)          │  Often an appliance
        │ single logical brain; HA cluster optional    │  too. Needs HA and a
        └──────────────────────────────────────────────┘  catalog backup.
                          │ duplication / replication
               ┌──────────┴───────────┐
               ▼                      ▼
        Cloud object tier      Tape library / second site / isolated vault
```

Key architectural points:

- **Brain:** a central primary server and catalog. Protect it with HA and catalog backups (Module 01).
- **Data path:** clients to appliances (media servers), using the LAN, SAN, or HotAdd paths from Module 02.
- **Deduplication:** performed in a **dedupe pool** on each media server or appliance. Data sent to different pools is typically not deduplicated against each other (Module 03).
- **Immutability and vaulting:** WORM storage on the appliance, plus replication to a second site, cloud, or tape (Module 04).
- **Scaling:** add media servers or appliances for throughput and capacity. The primary server scales up rather than out.

---

## Architecture B: Scale-Out Hyperconverged Clusters (Rubrik and Cohesity)

Both vendors use a **cluster of identical nodes** (compute, flash, and disk in each node) running a **distributed file system**. Backup software functions, metadata cataloging, media handling, and storage are converged into one platform, replacing the separate primary server, media servers, and storage of Architecture A.

```
  Clients / VMs / DBs ──(agent / API / hypervisor integration)──▶ cluster
  ┌─────────────────── SCALE-OUT CLUSTER (no master node) ───────────────────┐
  │                                                                           │
  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   + add nodes ▶     │
  │  │ NODE 1  │  │ NODE 2  │  │ NODE 3  │  │ NODE 4  │                      │
  │  │ compute │  │ compute │  │ compute │  │ compute │                      │
  │  │ flash+HD│  │ flash+HD│  │ flash+HD│  │ flash+HD│                      │
  │  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘                      │
  │  ═══════ DISTRIBUTED, IMMUTABLE FILE SYSTEM ═══════                      │
  │   Rubrik: Atlas      Cohesity: SpanFS                                     │
  │   distributed metadata / catalog · global dedupe · erasure coding         │
  └───────────────────────────────────┬───────────────────────────────────────┘
                                      ┊ replicate / archive
                         ┌────────────┴────────────┐
                         ▼                         ▼
                  Cloud archive            Isolated cyber vault / second cluster

  Management: SaaS control plane (Rubrik Security Cloud / Cohesity Helios)
```

**Rubrik building blocks**

- **Atlas:** distributed, immutable, versioned file system that stores the backup data.
- **Callisto:** distributed metadata system providing the global index and catalog.
- **Cerebro:** the data management layer that controls scheduling and data movement across the cluster.
- A zero-trust cluster design in which operations are performed only through authenticated APIs.

**Cohesity building blocks**

- **SpanFS:** distributed file system that provides linear scalability across hyperconverged nodes and rebalances data as nodes are added or removed.
- Clusters can start with as few as three nodes, with every node participating equally and no master node.
- **DataLock** for WORM protection, and **FortKnox** as an isolated cyber vault service.
- **Helios** as the central management plane across clusters.

Key architectural points:

- **Brain:** distributed across nodes. There is no separate primary server to fail.
- **Data path:** data is written into the cluster and spread across nodes.
- **Deduplication:** typically global across the cluster.
- **Immutability:** built into the file system and access model rather than added as an option.
- **Scaling:** add nodes, and capacity and performance usually grow together, with automatic rebalancing.

---

## Industry Note: Veritas NetBackup Is Now Part of Cohesity

Cohesity completed its acquisition of the Veritas data protection business, including NetBackup, in December 2024. Veritas Backup Exec, InfoScale, and Data Compliance were spun off into a separate company, Arctera. As a result:

- NetBackup is now a **Cohesity** product, although older documentation, installations, and colleagues still use the Veritas name.
- NetBackup can be managed from the Cohesity **Helios** control plane (available since August 2025).
- Cohesity has described a roadmap in which NetBackup data can be written to **SpanFS**, so the two architectures are gradually converging.
- NetBackup Flex Appliance releases continue, with the 8.0 release (October 2026) adding on-appliance malware scanning and IPsec protection for agentless connections.

The comparison in this module therefore describes two **architecture styles**, even though both may now come from the same vendor.

---

## Side-by-Side Comparison

| Aspect | NetBackup + Appliances | Rubrik / Cohesity scale-out |
|--------|------------------------|-----------------------------|
| Core design | Separate roles: primary server, media servers/appliances, clients | Converged cluster of identical nodes |
| Where the brain lives | Central primary server and catalog | Distributed across nodes |
| Scaling | Add media servers/appliances; primary server scales up | Add nodes; capacity and performance usually grow together |
| Single point of failure | Primary server and catalog need HA and catalog backup | No master node; cluster tolerates node failures |
| Deduplication scope | Typically per dedupe pool | Global across the cluster |
| Catalog / metadata | Database on the primary server | Distributed metadata across nodes |
| Immutability | WORM storage on appliance, object lock, hardened repositories | Immutable file system (Rubrik Atlas), DataLock WORM (Cohesity) |
| Vault / air gap | Replication to second site, cloud, tape, or vault design | Isolated vault services (for example FortKnox), replication, cloud |
| Management | NetBackup console; also manageable from Helios | SaaS control plane (Rubrik Security Cloud, Cohesity Helios) |
| Strengths | Very broad workload and platform support, flexibility, tape and legacy coverage | Simplicity, linear scale-out, built-in security and fast recovery |
| Trade-offs | More components to design, size, and tune | Node-based scaling ties capacity and compute; commitment to the vendor ecosystem |

---

## How to Choose

- **Large, mixed, long-established estate** (many workload types, tape, existing NetBackup investment, complex policies): NetBackup with Flex appliances is a natural evolution.
- **Greenfield or simplification project** (VMware-heavy, small operations team, desire for simple scaling and built-in ransomware features): a scale-out platform such as Rubrik or Cohesity.
- **Growing environment with a complexity problem:** evaluate consolidating onto appliances first, then compare with a scale-out migration based on workloads, skills, and cost.
- Always validate **workload support**, **recovery speed**, **immutability model**, and **total cost over the retention period** with a proof of concept.

## Common Pitfalls

- Comparing feature checklists instead of **architecture** (brain, data path, scaling).
- Assuming "no master node" removes the need for **vault copies and tested recovery**.
- Ignoring that **dedupe pools** in tiered designs can limit global deduplication efficiency.
- Sizing scale-out clusters only for capacity and forgetting **ingest performance** and **node count for resilience**.
- Treating a vendor claim of "immutable" as complete protection without checking credentials, access paths, and retention (Module 04).
- Forgetting that the **acquisition and roadmap** changes affect long-term planning for NetBackup environments.

---

## Whiteboard Checklist

- [ ] Draw two columns: tiered on the left, scale-out on the right
- [ ] Left: primary server, media servers/appliances, storage, cloud/tape
- [ ] Right: cluster of nodes over one distributed file system, plus vault/cloud
- [ ] Mark where the **catalog** and **dedupe** live in each
- [ ] Mark how each **scales** and what its **weak point** is
- [ ] Answer the three comparison questions: *Where is the brain? Where does the data land and who deduplicates it? How does it scale and what fails first?*

---

## Review Questions

1. A company runs a five-year-old NetBackup environment with one primary server, four media servers with separate dedupe pools, and a tape library. Data grows 30% per year and the team complains about complexity. Describe two possible ways forward and the trade-offs of each.
2. For both architectures, state where the control plane, the data plane, and the catalog live, and explain what happens when the "brain" fails in each design.

---

## Answers

**1. A five-year-old NetBackup environment (one primary server, four media servers with separate dedupe pools, a tape library) grows 30% per year and the team complains about complexity. What are two ways forward, and what are the trade-offs?**

**Option 1: Modernize within NetBackup using Flex appliances**

- Consolidate the media servers onto NetBackup Flex appliances, which combine primary, media, deduplication, and WORM storage in one turnkey system, and consolidate dedupe pools where possible.
- Keep the existing policies, catalog, workloads, skills, and tape library.
- **Advantages:** lower migration risk, no need to re-protect every workload, immutability added, and the change can be done in phases.
- **Trade-offs:** the design is still tiered. A central primary server remains (it still needs HA and catalog backups), dedupe pools may remain separate, and growth is handled by adding more appliances.

**Option 2: Move to a scale-out platform (Rubrik or Cohesity)**

- Replace the primary server, media servers, and storage with one cluster of nodes and add nodes as data grows.
- **Advantages:** simpler operations, one distributed file system with global deduplication, linear scaling, no separate master node, and built-in immutability.
- **Trade-offs:** a migration project; old NetBackup backups must stay restorable until their retention expires; workload and tape coverage must be validated; capacity and compute usually grow together; and it is a commitment to a new vendor ecosystem.

**Recommendation:** run a proof of concept for both. If the estate is mixed, includes tape, and has many workload types, start with Flex consolidation. If simplification is the main goal and workloads are mostly virtualized, evaluate scale-out. Because NetBackup is now part of Cohesity, a gradual path (manage NetBackup alongside Cohesity DataProtect in Helios) is also possible.

**2. Where do the control plane, data plane, and catalog live in each architecture, and what happens when the "brain" fails?**

| | NetBackup + Appliances | Scale-out cluster (Rubrik / Cohesity) |
|---|------------------------|---------------------------------------|
| Control plane | Primary server (policies, scheduling, orchestration) | Distributed across the cluster nodes |
| Data plane | Clients to media servers/appliances to storage; the primary server is not in the data path | Clients to the cluster nodes and the distributed file system |
| Catalog / metadata | Database on the primary server | Distributed metadata across the nodes |
| If the "brain" fails | Primary server down: no new jobs are scheduled and restores are hard to plan without the catalog. Data already stored on the media servers remains intact. | No single brain exists. If one node fails, the remaining nodes continue because data and metadata are protected across nodes. |
| Protection | HA cluster for the primary server, regular catalog backups stored off-site | Enough surviving nodes, plus replication to a second cluster, cloud, or vault |
| Remaining weak point | The primary server and catalog | Losing too many nodes or the whole site, so vault copies and tested recovery are still required |

---

## References

- Cohesity: [NetBackup Flex Appliances 8.0 solution brief](https://www.cohesity.com/resources/solution-brief/netbackup-flex-appliances-8-0/)
- Cohesity: [The future of NetBackup (white paper)](https://www.cohesity.com/content/dam/cohesity/resource-assets/white-paper/the-future-of-netbackup-en.pdf)
- CANCOM: [NetBackup was your choice and it remains so](https://www.cancom.at/en/blog/netbackup-was-your-choice-and-it-remains-so-what-is-changing-now-is-the-foundation)
- Cohesity: [What is a distributed file system (SpanFS)](https://www.cohesity.com/glossary/distributed-file-system/)
- Rubrik: [Introducing Atlas, Rubrik's Cloud-Scale File System](https://www.rubrik.com/en/blog/architecture/15/7/introducing-atlas-rubriks-cloud-scale-file-system.html)
- Rubrik: [Ransomware recovery: immutable backup architecture](https://www.rubrik.com/blog/architecture/20/3/ransomware-recovery-immutable-backup-architecture)

---

## Navigation

- Previous: [Module 04: Zero-Trust & Immutable Air-Gap Architectures](../04-immutable-air-gap-architectures/)
- Next: [Module 06: Architecture Challenges](../06-architecture-challenges/)
- Back to the [main README](../README.md)
