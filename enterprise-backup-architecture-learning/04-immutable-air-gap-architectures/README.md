# Module 04: Modern Zero-Trust & Immutable Air-Gap Architectures

> **Core principle:** assume the production network will be breached. Design the backup environment so that an attacker who controls production **still cannot delete, alter, or even reach** the last good copy of the data.

---

## Table of Contents

1. [Overview](#overview)
2. [Learning Objectives](#learning-objectives)
3. [Why Backups Are a Ransomware Target](#why-backups-are-a-ransomware-target)
4. [Immutability (WORM)](#immutability-worm)
5. [Air Gaps: Logical vs. Physical](#air-gaps-logical-vs-physical)
6. [Zero-Trust Principles for Backup](#zero-trust-principles-for-backup)
7. [Reference Architecture: Immutable Storage with Air-Gapped Vault](#reference-architecture-immutable-storage-with-air-gapped-vault)
8. [Ransomware Blocked at the Network Boundaries](#ransomware-blocked-at-the-network-boundaries)
9. [Recovery: The Clean Room](#recovery-the-clean-room)
10. [Common Pitfalls](#common-pitfalls)
11. [Whiteboard Checklist](#whiteboard-checklist)
12. [Review Questions](#review-questions)
13. [Answers](#answers)
14. [Navigation](#navigation)

---

## Overview

Modules 01 to 03 covered how backups are controlled, transported, and stored efficiently. This module addresses **survivability**: how to ensure backups remain usable after a ransomware attack, an insider threat, or a compromised administrator account.

Three building blocks are combined:

- **Immutability (WORM):** data that cannot be changed or deleted until its retention period ends.
- **Air gaps:** separation between production and the backup vault, either logical or physical.
- **Zero trust:** no implicit trust in networks, identities, or administrators.

## Learning Objectives

By the end of this module you will be able to:

- Explain why attackers target backups and how they do it
- Describe WORM immutability, including governance and compliance modes
- Distinguish logical air gaps from physical air gaps
- Draw an immutable-storage architecture with an isolated vault
- Trace a ransomware attack step by step and show where each stage is blocked

---

## Why Backups Are a Ransomware Target

Modern ransomware operators know that a good backup defeats their extortion. Before encrypting production, they commonly try to:

- Steal privileged credentials (often domain administrator accounts)
- Locate the backup server, repositories, and catalog
- Delete or encrypt backup data, snapshots, and the catalog
- Wait for days or weeks (dwell time) so that infected data ages into the backups

A traditional design, where backup servers and repositories are joined to the production domain and reachable over the production network with the same credentials, can be destroyed in the same attack as production.

---

## Immutability (WORM)

**WORM** means *Write Once, Read Many*. Once data is written, it cannot be modified or deleted until its **retention period** expires, even by an administrator.

| Mode | Who can remove the lock before expiry | Typical use |
|------|---------------------------------------|-------------|
| **Governance mode** | Specially privileged users or processes can override | Protection against accidents and ordinary users, with controlled flexibility |
| **Compliance mode** | **Nobody**, including administrators and the account owner, until retention expires | Strongest protection against ransomware and insider threats; regulatory retention |

Common implementations:

- Object storage with **Object Lock** (S3-compatible, on-premises or cloud)
- Backup appliances and dedupe systems with a **retention lock** feature
- **Hardened Linux repositories** using filesystem-level immutability
- **WORM tape** cartridges

Important considerations:

- Immutability depends on a **trusted clock**. If an attacker can alter system time, retention can be shortened, so secure time sources matter.
- Space cannot be reclaimed until retention expires, so capacity must be planned for the full immutable period.
- A retention period that is too short gives the attacker a window to wait it out.

---

## Air Gaps: Logical vs. Physical

An **air gap** keeps a copy of the data unreachable from the production environment.

| Type | How it works | Strength | Weakness |
|------|--------------|----------|----------|
| **Physical air gap** | No connection at all: tape ejected and stored, or disk physically disconnected | Strongest: an attacker cannot reach what is not connected | Slow to operate and restore, manual handling, media management |
| **Logical air gap** | A connection exists but is **normally closed**; it opens only briefly under tight control, for example a one-way, scheduled replication that the vault initiates, with separate credentials and no production domain trust | Automated and faster to restore | Relies on software and configuration, so it is only as strong as its access controls |

> **Test for a real air gap:** if the same administrator credentials used in production can open the gate or delete the copy, it is not an air gap.

---

## Zero-Trust Principles for Backup

- **Assume breach:** design as if production is already compromised.
- **Separate identity:** backup servers and the vault are **not** joined to the production domain, and use dedicated accounts.
- **Multi-factor authentication** on every backup console and administrative access.
- **Least privilege and role separation:** the person who runs backups cannot change retention or delete the vault.
- **Multi-person approval** (four-eyes) for destructive actions such as deleting data or shortening retention.
- **Network segmentation:** a dedicated backup network with strict firewall rules; no inbound path from production to the vault.
- **Encryption** in transit and at rest.
- **Monitoring and anomaly detection:** alert on mass deletions, unusual change rates, and failed logins.

---

## Reference Architecture: Immutable Storage with Air-Gapped Vault

```
 ┌───────────────── PRODUCTION NETWORK (assume breach) ────────────────────┐
 │   Servers / VMs / Databases ──────▶ Backup Server + Media Servers        │
 └────────────────────────────────────┬────────────────────────────────────┘
                                      ║ backup data (data plane)
 ═════════ BOUNDARY 1: FIREWALL / SEGMENTATION / SEPARATE IDENTITY ═════════
                                      ▼
 ┌───────────────── BACKUP NETWORK (own accounts, MFA) ────────────────────┐
 │   PRIMARY REPOSITORY                                                    │
 │   Immutable (WORM), short retention, fast restore                       │
 └────────────────────────────────────┬────────────────────────────────────┘
                                      ┊ replication: one-way, scheduled,
                                      ┊ vault PULLS the data
 ═════════ BOUNDARY 2: LOGICAL AIR GAP (link normally CLOSED) ══════════════
                                      ▼
 ┌───────────────── ISOLATED VAULT (no production domain) ─────────────────┐
 │   IMMUTABLE COPY                                                        │
 │   Compliance-mode lock, long retention, integrity checks and scanning   │
 └────────────────────────────────────┬────────────────────────────────────┘
                                      ┊ copy to offline media
 ═════════ BOUNDARY 3: PHYSICAL AIR GAP (media ejected, offline) ══════════
                                      ▼
 ┌─────────────────────────────────────────────────────────────────────────┐
 │   TAPE / OFFLINE DISK (stored off-site)                                 │
 └─────────────────────────────────────────────────────────────────────────┘

 Legend:  ║ = data plane    ┊ = controlled / scheduled link    ═ = boundary
```

How to read the layout:

- **Boundary 1** separates production from the backup network using firewalls, segmentation, and a separate identity domain.
- **Boundary 2** is a logical air gap. The link to the vault is normally closed and opens only for controlled replication.
- **Boundary 3** is a physical air gap. Offline media cannot be reached by any network attack.
- Each layer adds an independent barrier, so a single failure does not expose every copy.

---

## Ransomware Blocked at the Network Boundaries

```
 ATTACKER ACTION                          DEFENSE
 ───────────────                          ───────
 1. Infects a production server           Cannot be fully prevented.
                                          Assume breach.
        │
        ▼
 2. Spreads across the production         Segmentation and endpoint
    network                               detection limit the spread.
        │
        ▼
 3. Tries to reach the Backup Server      X BLOCKED: firewall boundary,
                                          separate identity (not in the
                                          production domain), MFA.
        │
        ▼
 4. Uses stolen admin credentials to      X BLOCKED: immutable repository
    delete or encrypt backups             rejects delete/modify commands
                                          until retention expires.
        │
        ▼
 5. Tries to reach the Vault              X BLOCKED: no route, link closed;
                                          vault pulls on schedule with its
                                          own credentials.
        │
        ▼
 6. Result: production data encrypted     OK: clean copy survives in the
                                          vault. Scan it, then restore in
                                          a clean room.
```

---

## Recovery: The Clean Room

Surviving backups are only useful if they can be restored **safely**:

1. **Declare the incident** and isolate affected systems.
2. **Select a restore point** from before the infection began. Dwell time means the most recent backup may already contain malware.
3. **Scan** the candidate backups for malware and indicators of compromise.
4. **Restore into an isolated clean room** network first and validate the systems.
5. **Rebuild trust:** rotate credentials and re-establish identity before reconnecting to production.

The extended backup guideline is **3-2-1-1-0**: three copies, on two media types, one off-site, one **immutable or offline**, and zero errors after recovery verification.

---

## Common Pitfalls

- **Backup servers joined to the production domain:** a single stolen domain admin account can reach everything.
- **"Air gap" that is always connected** with the same credentials. This is replication, not an air gap.
- **Immutable repository but unprotected catalog or backup server:** without the catalog, recovery is slow even when the data survives.
- **Retention set too short:** attackers can wait out the lock.
- **Governance mode assumed to equal compliance mode:** privileged accounts may still override governance locks.
- **Backing up already-infected data:** long dwell time means old restore points may be compromised. Keep enough history and scan before restoring.
- **Never testing recovery:** an untested vault is an assumption, not a plan.
- **Capacity not planned for immutability:** locked data cannot be deleted early to free space.

---

## Whiteboard Checklist

- [ ] Draw Production, Backup network, Vault, and Offline media as four stacked zones
- [ ] Mark three boundaries: firewall/identity, logical air gap, physical air gap
- [ ] Label the primary repository and the vault copy as immutable
- [ ] Draw the replication arrow as one-way, with the vault pulling
- [ ] Walk the attacker down the six steps and mark where each is blocked
- [ ] State clearly: **"If production credentials can delete it, it is not protected."**

---

## Review Questions

1. An attacker obtains domain administrator credentials and logs in to the backup server, then issues commands to delete all backups in the repository. Which controls would still protect the backup data, and why? Explain the difference between governance mode and compliance mode.
2. A vendor claims its backups are "air-gapped" because they are replicated to a second data center over an always-on link, using the same administrator credentials as production. Evaluate this claim and describe what you would change.

---

## Answers

**1. An attacker with domain administrator credentials logs in to the backup server and deletes all backups in the repository. What still protects the data, and what is the difference between governance and compliance mode?**

The backup data is still protected by several independent controls:

- **Immutable (WORM) repository:** the storage rejects delete and modify commands until the retention period expires, even when the request comes from an administrator.
- **Separate identity and MFA:** the backup environment and the vault are not joined to the production domain, so stolen production credentials do not give access to them.
- **Logical air-gapped vault:** the vault keeps its own immutable copy, its link to the backup network is normally closed, and the vault pulls data on a schedule with its own credentials.
- **Offline copy:** a tape or offline-disk copy cannot be reached over any network.

**Governance vs. compliance mode:**

- **Governance mode:** specially privileged users can override the lock before retention ends. It protects against accidents and ordinary users, but a compromised privileged account could remove it.
- **Compliance mode:** nobody, including administrators and the account owner, can remove the lock until retention ends. It is the stronger choice against ransomware and insider threats.

**2. A vendor calls its backups "air-gapped" because they replicate to a second data center over an always-on link using the same administrator credentials as production. Is the claim valid?**

No. This is **replication**, not an air gap.

- The link is always open, so ransomware or an attacker can travel across it.
- The same credentials are used, so an attacker who steals production administrator access can reach and delete the remote copy as well.
- A second data center protects against a site failure, but not against a compromised account.

What to change:

- Make the link **normally closed** and one-way, with the vault **pulling** data on a schedule.
- Use **separate credentials and identity** for the vault, with MFA and multi-person approval for destructive actions.
- Make the vault copy **immutable** (compliance mode) with a long enough retention period.
- Keep an **offline copy** (for example tape) for a physical air gap.
- Test the rule: **if production credentials can delete it, it is not an air gap.**

---

## Navigation

- Previous: [Module 03: Storage Targets & Deduplication](../03-storage-targets-deduplication/)
- Next: [Module 05: Vendor Architecture Comparisons](../05-vendor-architecture-comparisons/)
- Back to the [main README](../README.md)
