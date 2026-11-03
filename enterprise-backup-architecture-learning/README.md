# Enterprise Backup Architecture: Learning Path

A hands-on, diagram-first learning path covering **enterprise backup architecture and data flow**, from absolute zero to expert level. Every module is built around text-based visual architectures (ASCII diagrams, component maps, flowcharts) so each concept can be redrawn on a whiteboard and explained out loud.

> **Core principle:** the system that decides what to do (control plane) is not the system that moves the data (data plane).

---

## Who this is for

- System / Server engineers moving into backup and data protection
- DevOps and cloud engineers who need to understand how enterprise backup is wired
- Anyone who has to design, review, or explain a backup architecture

No prior backup experience is assumed.

## What you will be able to do

- Explain the Master / Media / Client tiers and draw the control plane vs. data plane
- Trace how data moves from a production database to a storage target
- Choose between LAN, LAN-free (SAN), and HotAdd data paths
- Compare source-side and target-side deduplication
- Design immutable, air-gapped backup vaults that survive ransomware
- Compare traditional (NetBackup-style) and scale-out (Rubrik/Cohesity-style) architectures
- Solve real design scenarios and defend your choices

---

## Modules

| # | Module | Topics | Status |
|---|--------|--------|--------|
| 01 | [Three-Tier Enterprise Backup Topology](./01-three-tier-backup-topology/) | Master/Media/Client roles, control vs. data plane, database-to-storage data flow | Available |
| 02 | [Network Topologies & Data Paths](./02-network-topologies-data-paths/) | LAN (NBD), SAN / LAN-free, HotAdd for VMs, side-by-side pipeline maps | Planned |
| 03 | [Storage Targets & Deduplication](./03-storage-targets-deduplication/) | Source-side vs. target-side dedupe, hash/segment-DB workflow, SSD / dedupe appliances / cloud object / tape | Planned |
| 04 | [Zero-Trust & Immutable Air-Gap Architectures](./04-immutable-air-gap-architectures/) | WORM, logical vs. physical air-gap, ransomware boundary diagrams | Planned |
| 05 | [Vendor Architecture Comparisons](./05-vendor-architecture-comparisons/) | Traditional (Veritas NetBackup + Appliances) vs. scale-out (Rubrik / Cohesity) | Planned |
| 06 | [Architecture Challenges](./06-architecture-challenges/) | Three design scenarios, critique, and reference solutions | Planned |

---

## Repository structure

```
enterprise-backup-architecture/
├── README.md
├── 01-three-tier-backup-topology/
│   └── README.md
├── 02-network-topologies-data-paths/
│   └── README.md
├── 03-storage-targets-deduplication/
│   └── README.md
├── 04-immutable-air-gap-architectures/
│   └── README.md
├── 05-vendor-architecture-comparisons/
│   └── README.md
├── 06-architecture-challenges/
│   └── README.md
└── glossary.md            (optional: terms used across modules)
```

Each module folder contains one `README.md` with the lesson, diagrams, and review questions. Add extra files (cheat sheets, lab notes, whiteboard photos) next to it as you go.

---

## How to use this repository

1. Read the modules **in order**. Each one builds on the previous.
2. **Redraw every diagram by hand** before moving on. If you can't draw it from memory, you don't own it yet.
3. Answer the review questions in your own words, then compare with the reference answers.
4. Finish with Module 06 and design the scenarios without looking at the solutions.

## Quick glossary (Module 01 preview)

| Term | Meaning |
|------|---------|
| **Master / Management Server** | The brain: policies, scheduler, catalog, job tracking |
| **Media / Storage Node** | The muscle: receives data, compresses, encrypts, dedupes, writes to storage |
| **Client / Agent** | Software on the production host that reads data and keeps it application-consistent |
| **Control plane** | Small command and metadata traffic ("start job", "job done") |
| **Data plane** | The bulk backup payload moving across the network |
| **Catalog** | Metadata database recording what was backed up, where, and for how long |

---

## Vendor-neutral note

Vendor names (Veritas NetBackup, Commvault, Veeam, Dell NetWorker, Rubrik, Cohesity, Dell Data Domain) are used as examples of architectural patterns. Terminology and ports differ by product and version, so always confirm details against the official documentation for the version you run.

## Contributing

Spotted a mistake or have a better diagram? Open an issue or a pull request.

## License

Choose a license (for example MIT for code samples and CC BY 4.0 for written content) and add a `LICENSE` file.

## Author

**Emdadul Haque Talha**
BSc in CSE | Intern Engineer, Systems / Servers
Learning path: system and server engineering, with a possible future toward cloud and DevOps.
