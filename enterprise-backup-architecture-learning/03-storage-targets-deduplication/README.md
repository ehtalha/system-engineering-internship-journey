# Module 03: Storage Targets & Deduplication Architecture

> **Core principle:** backup data is highly repetitive. Deduplication stores each unique piece of data **once** and replaces every repeat with a small pointer, which is what makes long retention affordable.

---

## Table of Contents

1. [Overview](#overview)
2. [Learning Objectives](#learning-objectives)
3. [What Deduplication Is](#what-deduplication-is)
4. [How a Data Block Is Deduplicated](#how-a-data-block-is-deduplicated)
5. [Source-Side vs. Target-Side Deduplication](#source-side-vs-target-side-deduplication)
6. [Inline vs. Post-Process Deduplication](#inline-vs-post-process-deduplication)
7. [Physical Storage Tiers](#physical-storage-tiers)
8. [Storage Tier Comparison](#storage-tier-comparison)
9. [Common Pitfalls](#common-pitfalls)
10. [Whiteboard Checklist](#whiteboard-checklist)
11. [Review Questions](#review-questions)
12. [Answers](#answers)
13. [Navigation](#navigation)

---

## Overview

Modules 01 and 02 explained *who* controls a backup and *how* the data travels. This module explains **where the data finally lands** and **how it is stored efficiently**.

Two ideas are covered:

- **Deduplication:** reducing the amount of data stored and transferred by eliminating repeated chunks.
- **Storage tiers:** the physical targets (flash, dedupe appliances, cloud object storage, tape) and the role each one plays.

## Learning Objectives

By the end of this module you will be able to:

- Explain chunking, hashing, and the segment index
- Draw the deduplication workflow for a single data block
- Compare source-side and target-side deduplication, and inline and post-process designs
- Describe the four storage tiers and when each is used
- Identify what reduces or destroys deduplication efficiency

---

## What Deduplication Is

Consider a nightly full backup of a 10 TB database where only about 2% of the data changes each day. Each night's full backup is roughly 98% identical to the previous one.

Illustrative arithmetic for 30 nightly full backups:

| Approach | Data stored |
|----------|-------------|
| Without deduplication | 30 x 10 TB = **300 TB** |
| With deduplication (first full + about 0.2 TB of new data per day) | 10 TB + (29 x 0.2 TB) = about **15.8 TB** |
| Resulting ratio | roughly **19 : 1** |

Real ratios depend on the data type, change rate, and retention. The figures above are for illustration only.

**Analogy:** an office has 30 employees, and each is given the same 100-page handbook. Instead of printing 30 copies, you keep one master copy and give each employee a reference slip. If one page changes, only the new page is stored.

---

## How a Data Block Is Deduplicated

```
   Backup stream (for example, 10 TB)
              │
              ▼
 ┌─────────────────────────┐
 │ 1. CHUNKING             │  Split the stream into segments
 │    fixed or variable    │  (a few KB to 128 KB, product dependent)
 └────────────┬────────────┘
              ▼
 ┌─────────────────────────┐
 │ 2. HASHING              │  Fingerprint = cryptographic hash of the chunk
 │    chunk -> fingerprint │  Same data always gives the same fingerprint
 └────────────┬────────────┘
              ▼
 ┌─────────────────────────┐        ┌───────────────────────────┐
 │ 3. INDEX LOOKUP         │◀──────▶│ SEGMENT INDEX (database)  │
 │    "seen this before?"  │        │ fingerprint -> location   │
 └──────┬───────────┬──────┘        │ (kept in RAM / SSD)       │
        │ YES       │ NO            └───────────────────────────┘
        ▼           ▼
 ┌─────────────┐ ┌───────────────────────────┐
 │ 4a DUPLICATE│ │ 4b NEW CHUNK              │
 │ store only  │ │ compress -> (encrypt) ->  │
 │ a POINTER   │ │ write to a disk container │
 │ (no data)   │ │ add fingerprint to index  │
 └──────┬──────┘ └─────────────┬─────────────┘
        └──────────┬───────────┘
                   ▼
 ┌─────────────────────────────────────┐
 │ 5. UPDATE THE FILE RECIPE           │  Ordered list of fingerprints:
 │    (metadata: how to rebuild)       │  the map a restore will follow
 └─────────────────────────────────────┘
```

| Step | What happens |
|------|--------------|
| 1. Chunking | The stream is split into segments. **Variable-length** chunking tolerates inserted data better than fixed-length chunking. |
| 2. Hashing | Each chunk is reduced to a fingerprint using a cryptographic hash. |
| 3. Index lookup | The fingerprint is checked against the segment index to see whether the chunk already exists. |
| 4a. Duplicate | Only a pointer to the existing chunk is stored. |
| 4b. New chunk | The chunk is compressed (and optionally encrypted) and written to disk, and its fingerprint is added to the index. |
| 5. File recipe | Metadata records the ordered list of fingerprints so the original data can be rebuilt. |

**Restore ("rehydration"):** the system follows the recipe, fetches each referenced chunk, and reassembles the original data. Heavily fragmented chunk layouts can make restores slower than backups.

> **Order matters:** deduplicate first, then compress, then encrypt. Encrypting before deduplication makes identical data look different and destroys the ratio.

---

## Source-Side vs. Target-Side Deduplication

```
 A) SOURCE-SIDE                              B) TARGET-SIDE
 ───────────────                             ───────────────
 ┌──────────────────────┐                    ┌──────────────────────┐
 │ CLIENT / MEDIA SERVER│                    │ CLIENT / MEDIA SERVER│
 │ 1. chunk             │                    │ reads data and sends │
 │ 2. hash              │                    │ EVERYTHING           │
 │ 3. ask target:       │                    └──────────┬───────────┘
 │   "have this hash?"  │                               ║
 └─────────┬────────────┘                               ║ 100% of the data
           ┊ small hash queries                         ║ crosses the network
           ┊ + ONLY unique chunks                       ▼
           ▼                                  ┌──────────────────────┐
 ┌──────────────────────┐                    │ DEDUPE TARGET        │
 │ DEDUPE TARGET        │                    │ 1. chunk 2. hash     │
 │ (segment index)      │                    │ 3. check 4. store    │
 └──────────────────────┘                    │    unique only       │
                                             └──────────────────────┘
 Network load: LOW   Client CPU: HIGHER      Network load: HIGH  Client CPU: LOW
```

| Aspect | Source-side | Target-side |
|--------|-------------|-------------|
| Where hashing happens | On the client or media server | On the storage target |
| Data sent over the network | Only unique chunks | Everything |
| Network usage | Low | High |
| CPU load on client | Higher | Low |
| Best for | WAN / branch offices, constrained networks, large nightly volumes | Simple environments, clients that should not carry extra CPU load |
| Examples | Client-side deduplication in NetBackup and Commvault; Data Domain DD Boost (distributed segment processing) | Backup written to a dedupe appliance without client-side processing |

## Inline vs. Post-Process Deduplication

| Aspect | Inline | Post-process |
|--------|--------|--------------|
| When deduplication happens | Before data is written to disk | After data lands on a disk **landing zone** |
| Disk space needed | Only the deduplicated size | Extra landing-zone capacity |
| Ingest speed | Depends on CPU and index speed | Fast, because data is written first |
| Recent-restore speed | Needs rehydration | Latest backup often kept in full on the landing zone, so restores can be faster |
| Typical examples | Dell Data Domain (inline) | Landing-zone designs such as ExaGrid |

Many modern platforms combine a fast flash tier for ingest with deduplicated storage behind it.

---

## Physical Storage Tiers

```
   PRODUCTION DATA
         │
         ▼
 ┌─────────────────────────────────────────────┐
 │ TIER 1: FLASH / SSD                         │  Fast ingest, recent backups,
 │ (landing zone, cache, index, metadata)      │  instant recovery, dedupe index
 └───────────────────────┬─────────────────────┘
                         ▼
 ┌─────────────────────────────────────────────┐
 │ TIER 2: DEDUPLICATION APPLIANCE (disk)      │  Days to months of retention,
 │ for example Dell Data Domain                │  high deduplication ratio
 └───────────────────────┬─────────────────────┘
              ┌──────────┴───────────┐
              ▼                      ▼
 ┌─────────────────────────┐ ┌─────────────────────────┐
 │ TIER 3: CLOUD OBJECT    │ │ TIER 3: TAPE LIBRARY    │
 │ STORAGE (S3, Blob, etc.)│ │                         │
 │ Long retention, off-site│ │ Offline, lowest cost/TB │
 │ elastic capacity        │ │ natural air gap         │
 └─────────────────────────┘ └─────────────────────────┘
```

- **Flash / SSD:** absorbs high write speed, holds the dedupe index and metadata, and supports fast recovery of recent backups.
- **Deduplication appliance:** purpose-built disk system (for example Dell Data Domain) that stores deduplicated data efficiently for short and medium retention.
- **Cloud object storage:** durable, elastic, and off-site, well suited for long retention. Consider egress charges and restore latency.
- **Tape library:** very low cost per terabyte and offline when ejected, suitable for archival and as a natural air gap. Restores are slow and sequential.

Not every environment uses all four tiers. The common guideline is the **3-2-1 rule**: three copies of data, on two different media types, with one copy off-site.

## Storage Tier Comparison

| Feature | Flash / SSD | Dedupe appliance | Cloud object | Tape |
|---------|-------------|------------------|--------------|------|
| Speed | Fastest | Fast | Variable (network dependent) | Slowest, sequential |
| Cost per TB | Highest | Medium | Low to medium (plus egress) | Lowest |
| Typical retention | Days | Days to months | Months to years | Years |
| Restore time | Fastest | Fast | Slower | Slowest |
| Main role | Ingest and instant recovery | Primary backup repository | Long-term, off-site copy | Archive, offline copy |

---

## Common Pitfalls

- **Encrypting or compressing before deduplication:** identical data stops looking identical and the ratio collapses.
- **Data that is already compressed or encrypted** (media files, some encrypted database features) deduplicates poorly.
- **Undersized segment index memory:** the index must be fast. If it spills out of RAM or SSD, ingest slows sharply.
- **Fragmentation:** after many generations, restores can slow down because chunks are scattered.
- **Cleaning and garbage collection:** freed space is reclaimed only after a cleaning cycle, so capacity numbers can lag behind.
- **Single dedupe domain:** if the appliance or its index is lost, every backup that depends on it is affected. Replicate to a second system.
- **Quoted ratios:** vendor ratios are workload dependent. Size using measured change rates, not marketing numbers.

---

## Whiteboard Checklist

- [ ] Draw the five steps: chunk, hash, index lookup, duplicate vs. new, file recipe
- [ ] Draw the segment index as a separate box connected to the lookup step
- [ ] Draw source-side and target-side side by side, marking where hashing happens and what crosses the network
- [ ] Draw the four storage tiers from fastest to cheapest
- [ ] State clearly: **"Deduplicate first, then compress, then encrypt."**

---

## Review Questions

1. A branch office must back up 2 TB nightly to a central site over a 100 Mbps WAN link. Which deduplication architecture would you choose and why? State one trade-off.
2. After a database team enables encryption on the data before it reaches the backup system, the deduplication ratio drops from about 15:1 to about 1.5:1. Explain why this happens and what you would do about it.

---

## Answers

**1. A branch office must back up 2 TB nightly over a 100 Mbps WAN link. Which deduplication architecture, and what is one trade-off?**

**Source-side deduplication** is the appropriate choice.

- A 100 Mbps link carries roughly 12.5 MB/s, or about 45 GB per hour in theory. Sending 2 TB (about 2,048 GB) takes roughly **45 hours**, so the backup cannot complete within a nightly window.
- With source-side deduplication, the client chunks and hashes the data, asks the target which hashes already exist, and sends **only the unique chunks**. If 2% to 5% of the data changes, only about 40 to 100 GB crosses the WAN, which takes roughly 1 to 2.5 hours at the theoretical rate.
- **Trade-off:** hashing consumes CPU and memory on the client, so an agent is required and production performance may be affected. Schedule backups outside peak hours or use a local media server at the branch.
- Note: the very first full backup is still large. Seed it locally or by another method rather than sending it over the WAN.

**2. Why did the deduplication ratio fall from about 15:1 to about 1.5:1 after the database team enabled encryption?**

The data now reaches the backup system **already encrypted**.

- Encryption makes identical plaintext blocks appear as different ciphertext (for example through unique initialization vectors or nonces), so their fingerprints no longer match. The deduplication engine finds almost no duplicates.
- Encrypted data is also effectively incompressible, so compression gains disappear as well.
- The backup system's own processing order (deduplicate, then compress, then encrypt) is correct and is **not** the cause. The problem is encryption applied **before** deduplication.

Recommended actions:

- Where policy allows, move encryption to a point **after** deduplication: encrypt in the backup software after dedupe, or use encryption at rest on the storage target, with TLS for data in transit.
- If database-level encryption is mandatory, plan for a very low deduplication ratio and size capacity from measured results rather than the previous 15:1 figure.
- Measure the ratio in a pilot before rollout, and coordinate the change with the security team.

---

## Navigation

- Previous: [Module 02: Network Topologies & Data Paths](../02-network-topologies-data-paths/)
- Next: [Module 04: Zero-Trust & Immutable Air-Gap Architectures](../04-immutable-air-gap-architectures/)
- Back to the [main README](../README.md)
