# Module 01: The Three-Tier Enterprise Backup Topology

> **Core principle:** the system that decides *what to do* is not the system that *moves the data*.

---

## Table of Contents

1. [Overview](#overview)
2. [Learning Objectives](#learning-objectives)
3. [The Three Tiers](#the-three-tiers)
4. [Control Plane vs. Data Plane](#control-plane-vs-data-plane)
5. [Data Flow: Production Database to Storage Target](#data-flow-production-database-to-storage-target)
6. [Engineering Takeaways](#engineering-takeaways)
7. [Whiteboard Checklist](#whiteboard-checklist)
8. [Review Questions](#review-questions)
9. [Navigation](#navigation)

---

## Overview

Every enterprise backup product, including Veritas NetBackup, Commvault, Veeam, and Dell NetWorker, is built on the same separation of responsibilities. Vendors use different names, but the roles are identical: a **management tier** that makes decisions, a **data mover tier** that handles the payload, and a **client tier** that is the source of the data.

Understanding this separation is the foundation for every later module (network paths, deduplication, immutability, and vendor comparisons).

## Learning Objectives

By the end of this module you will be able to:

- Explain the role of the Master/Management Server, Media/Storage Nodes, and Client Agents
- Draw the **control plane** and **data plane** on a whiteboard and explain the difference
- Trace a production database backup from trigger to catalog update
- Identify the bottlenecks, scaling model, and failure domains of each tier

---

## The Three Tiers

| Role | Responsibility | Veritas | Commvault | Veeam | Dell NetWorker |
|------|----------------|---------|-----------|-------|----------------|
| **Management tier** (the brain) | Policies, schedules, catalog, job tracking, reporting | Master Server | CommServe | Backup Server | NetWorker Server |
| **Data mover tier** (the muscle) | Receives the payload, processes it, writes to storage | Media Server | MediaAgent | Proxy + Repository | Storage Node |
| **Client tier** (the source) | Reads production data and hands it off | Client / agent | File System / DB agent | Agent / hypervisor integration | Client |

### Tier 1: Master / Management Server

- Holds the **catalog**, a database recording which files, blocks, or databases are in which backup image, on which storage, with which retention. Without the catalog, data exists but cannot be located or restored.
- Runs the scheduler, policy engine, and job manager.
- Authenticates and authorizes all components.
- Normally does **not** touch backup payload. If it does, treat that as a design smell.

### Tier 2: Media / Storage Nodes

- Receive the data stream from clients.
- Perform compression, encryption, deduplication, and multiplexing.
- Write to the storage target (disk, dedupe appliance, cloud, tape).
- Are the **throughput bottleneck** in most designs. Scale them out to scale backup bandwidth.

### Tier 3: Client Agents

- Small software component on the production host (or an integration with the hypervisor).
- Talks to the application so the backup is **application-consistent**. For example, it uses SQL VDI/VSS or Oracle RMAN instead of copying raw files mid-write.
- Reads data, optionally performs source-side work (such as deduplication, covered in Module 03), and sends it to the media server.

---

## Control Plane vs. Data Plane

- **Control plane:** small messages such as "start job", "I'm ready", "job done", and catalog updates. Low bandwidth, latency-sensitive. Needs reliable name resolution, firewall rules, and certificates.
- **Data plane:** the actual backup payload. Huge, sustained, and bandwidth-hungry. This is where NICs and HBAs get saturated.

```
                       ┌─────────────────────────────────┐
                       │   TIER 1: MASTER / MGMT SERVER  │
                       │   ┌───────────┐ ┌─────────────┐ │
                       │   │ Scheduler │ │   Catalog   │ │
                       │   │  Policies │ │  (metadata) │ │
                       │   └───────────┘ └─────────────┘ │
                       └───────┬─────────────────┬───────┘
                               ┊                 ┊
          CONTROL PLANE        ┊                 ┊       CONTROL PLANE
      (small commands, 1-way   ┊                 ┊     (tell media server:
       "start backup now")     ┊                 ┊      "expect client X,
                               ┊                 ┊       write to pool Y")
                               ▼                 ▼
   ┌──────────────────────────────┐     ┌──────────────────────────────┐
   │  TIER 3: CLIENT + AGENT      │     │  TIER 2: MEDIA / STORAGE     │
   │  ┌────────────────────────┐  │     │  NODE                        │
   │  │  Production Database   │  │     │  ┌────────────────────────┐  │
   │  │  (SQL / Oracle / etc.) │  │     │  │ Receive → Compress →   │  │
   │  └───────────┬────────────┘  │     │  │ Encrypt → Dedupe →     │  │
   │              │ app-consistent│     │  │ Write                  │  │
   │  ┌───────────▼────────────┐  │     │  └───────────┬────────────┘  │
   │  │  Backup Agent          │══╪═════╪═══▶ (inbound)│               │
   │  └────────────────────────┘  │     │              ▼               │
   └──────────────────────────────┘     └──────────────┬───────────────┘
                                                       ║
                  ┊┊┊  = CONTROL PLANE (thin)          ║ DATA PLANE
                  ═══▶ = DATA PLANE (thick, bulk)      ║ (media server → target)
                                                       ▼
                                      ┌──────────────────────────────┐
                                      │  STORAGE TARGET              │
                                      │  Disk / Dedupe Appliance /   │
                                      │  Cloud Object / Tape         │
                                      └──────────────────────────────┘

   Metadata path (small, goes back UP to the master):
   Media Server ┈┈┈▶ Master Server  ("image X written, 1.2 TB, expires in 30 days")
```

**How to read the diagram**

- Dotted vertical lines from the master are the **control plane**. They carry instructions, not payload.
- Thick double lines from client to media server and from media server to storage are the **data plane**. The master is not in this path.
- The **catalog update** returns to the master after the data lands. This is the metadata return path.

---

## Data Flow: Production Database to Storage Target

Full lifecycle of a nightly SQL Server backup:

```
 TIME ──────────────────────────────────────────────────────────────────▶

 MASTER          CLIENT (SQL host)         MEDIA SERVER         STORAGE
   │                    │                        │                  │
   │ ① Schedule fires   │                        │                  │
   │ (policy: 02:00)    │                        │                  │
   │                    │                        │                  │
   │ ② "Start job" ┈┈┈▶ │                        │                  │
   │ ② "Expect client" ┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈▶  │                  │
   │                    │                        │                  │
   │                    │ ③ Agent calls DB API   │                  │
   │                    │    (VDI/VSS/RMAN)      │                  │
   │                    │    → consistent point  │                  │
   │                    │                        │                  │
   │                    │ ④ Open data connection │                  │
   │                    │ ═══════════════════▶   │                  │
   │                    │    stream payload      │                  │
   │                    │                        │ ⑤ Compress /     │
   │                    │                        │   encrypt /      │
   │                    │                        │   dedupe         │
   │                    │                        │                  │
   │                    │                        │ ⑥ Write ════════▶│
   │                    │                        │                  │
   │                    │ ⑦ "Stream complete"    │                  │
   │                    │ ═══════════════════▶   │                  │
   │                    │                        │                  │
   │ ⑧ Catalog update ◀┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈ │                  │
   │   (image ID, size, │                        │                  │
   │    location,       │                        │                  │
   │    retention)      │                        │                  │
   │                    │ ⑨ DB released from     │                  │
   │                    │    backup state        │                  │
   │ ⑩ Job = SUCCESS    │                        │                  │
```

| Step | What happens |
|------|--------------|
| 1 | **Trigger.** The master's scheduler fires per policy (what, when, how long to keep, where to store). |
| 2 | **Orchestration.** The master tells the client to begin and tells the media server what is coming. |
| 3 | **Quiesce.** The agent asks the database for a consistent stream or snapshot through its native backup API. |
| 4 | **Connection.** The client opens a direct network connection to the media server. The payload does not pass through the master. |
| 5 | **Processing.** The media server compresses, encrypts, and deduplicates (unless the client already did part of this). |
| 6 | **Write.** The media server writes to the storage target. |
| 7 | **Completion.** The client signals that the stream is finished. |
| 8 | **Catalog.** The media server reports back, and the master records the image in the catalog. |
| 9 | **Release.** The database leaves backup mode. |
| 10 | **Job status.** The master marks the job successful, partial, or failed. |

---

## Engineering Takeaways

### Where bottlenecks live

| Component | Typical limits |
|-----------|----------------|
| Client | Disk read speed, CPU for agent-side work, NIC |
| Network | Link between client and media server (the usual limit in LAN backups) |
| Media server | CPU for dedupe/compression/encryption, NIC, write speed to storage |
| Storage target | Ingest rate |

### Scaling model

- The **master** scales **up** (typically a single protected node, often clustered for high availability).
- **Media servers** scale **out** (add more to raise aggregate throughput).
- **Clients** scale by count and are managed through policy.

### Failure domains

- **Master down:** no new jobs run and restores cannot be planned. Back up the **catalog** itself; it is the backup of the backup system.
- **Media server down:** jobs fail over to another media server, if designed for it.
- **Client agent broken:** only that host is unprotected.

### Network and firewall view

- **Control plane:** master to clients and master to media servers. Usually fixed ports, with certificate-based authentication in modern versions.
- **Data plane:** client to media server, and media server to storage. Often placed on a **dedicated backup network or VLAN** so backup traffic does not compete with production traffic.

---

## Whiteboard Checklist

- [ ] Draw the three boxes: Master, Media Server, Client
- [ ] Add thin dotted lines for the control plane
- [ ] Add thick lines for the data plane (client to media server to storage)
- [ ] Draw the catalog metadata return path to the master
- [ ] State clearly: **"The master never touches the payload."**

---

## Review Questions

1. A colleague says, "Our backups are slow, so let's upgrade the master server's CPU and RAM." Using the control plane / data plane model, explain whether this is likely to help. Where would you look first for the bottleneck?
2. What happens to your ability to **restore** if the master server and its catalog are lost but all backup data on the storage target is intact? Which design practice protects you from this?

Answers

1. Will upgrading the master server's CPU and RAM fix slow backups?

Unlikely. The master only handles the control plane (scheduling, policies, catalog). It does not touch the backup payload, so slow backups are almost always a data plane problem.

Measure along the data path and find the slowest stage:

Client (disk read, CPU, NIC) → Network link → Media Server (CPU, RAM, NIC) → Storage (ingest rate)

Check the master last, only if jobs are slow to start or a job contains millions of small files (heavy catalog load).

2. What if the master and its catalog are lost but the backup data is intact?

The data is safe but effectively unreachable. The catalog is the map that records which image is on which storage and how long it is kept, so you cannot easily locate or restore anything. Rebuilding the catalog by scanning media is slow and may be incomplete.

Protection:
 1. Back up the catalog regularly and automatically (for example, the NetBackup catalog backup / DR package)
 2. Store a copy in a separate failure domain (off-site or a different system), not on the same storage as the backups
 3. Run the master in an HA cluster where possible
 4. Test catalog restores periodically
---

## Navigation

- Previous: none (this is the first module)
- Next: [Module 02: Network Topologies & Data Paths](../02-network-topologies-data-paths/)
- Back to the [main README](../README.md)
