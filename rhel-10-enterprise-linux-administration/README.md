# Red Hat Enterprise Linux (RHEL) 10.2: Hands-On Administration Journey

A structured, module-by-module record of learning **Red Hat Enterprise Linux 10.2** from scratch, practiced hands-on in a virtual machine and documented as part of my **System/Server Engineer internship**.

Each module contains the concepts, the exact commands I ran, real terminal output, troubleshooting notes and key takeaways, so that anyone starting from zero can follow the same path.

---

## Table of Contents

1. [About This Repository](#about-this-repository)
2. [Lab Environment](#lab-environment)
3. [Curriculum and Progress](#curriculum-and-progress)
4. [Repository Structure](#repository-structure)
5. [How Each Module Is Documented](#how-each-module-is-documented)
6. [Conventions](#conventions)
7. [References](#references)
8. [Author](#author)
9. [Disclaimer](#disclaimer)

---

## About This Repository

| | |
|---|---|
| **Goal** | Build production-ready Linux administration skills, from fundamentals to enterprise storage, security and recovery |
| **Audience** | Beginners moving into system, server, storage, cloud or DevOps engineering |
| **Approach** | One topic at a time: concept, hands-on lab, verification, documentation, then a self-quiz with scenario questions |
| **Platform** | RHEL 10.2 |

### Learning Objectives

By the end of this journey I should be able to:

- Navigate and administer a RHEL server entirely from the command line
- Manage users, groups, permissions and access control
- Install, update and audit software with DNF and RPM
- Design and manage storage with partitions, filesystems and LVM
- Control services and boot behavior with systemd
- Configure networking, firewalld and SELinux securely
- Troubleshoot common production issues and recover a failed system
- Apply fundamental backup and recovery techniques

---

## Lab Environment

| Component | Details |
|---|---|
| Hypervisor | VMware Workstation |
| Guest OS | Red Hat Enterprise Linux 10.2 |
| Access | VM console and SSH |
| Package source | Red Hat subscription (Developer Subscription) |

> Verify your own system with:
>
> ```bash
> cat /etc/os-release
> uname -r
> ```

---

## Curriculum and Progress

| # | Module | Key Topics | Status |
|---|--------|-----------|--------|
| 01 | [Basics](01-basics/) | RHEL identity, shell, navigation, file operations, File System Hierarchy, built-in help | In progress |
| 02 | [Users and Groups](02-users-groups/) | User and group management, password policy, sudo | Not started |
| 03 | [Permissions](03-permissions/) | Ownership, rwx, special bits, umask, ACLs | Not started |
| 04 | [Software Management](04-dnf-rpm/) | Subscription, DNF, RPM, repositories, updates | Not started |
| 05 | [Storage and LVM](05-storage-lvm/) | Disks, partitions, filesystems, LVM, fstab, mounts | Not started |
| 06 | [systemd](06-systemd/) | Units, services, targets, journald | Not started |
| 07 | [Networking](07-networking/) | NetworkManager, nmcli, DNS, SSH | Not started |
| 08 | [Firewalld](08-firewalld/) | Zones, services, ports, rich rules | Not started |
| 09 | [SELinux](09-selinux/) | Modes, contexts, booleans, troubleshooting denials | Not started |
| 10 | [Processes and Scheduling](10-processes-scheduling/) | Process control, cron, systemd timers | Not started |
| 11 | [Troubleshooting and Recovery](11-troubleshooting-recovery/) | Boot process, rescue mode, log analysis | Not started |
| 12 | [Backup and Recovery](12-backup-recovery/) | tar, rsync, LVM snapshots, restore testing | Not started |

**Legend:** In progress = currently studying. Not started = planned. Completed modules are marked **Completed**.

---

## Repository Structure

```text
rhel-10-enterprise-linux-administration/
├── README.md                      # This file
├── 01-basics/
│   └── README.md
├── 02-users-groups/
│   └── README.md
├── 03-permissions/
│   └── README.md
├── 04-dnf-rpm/
│   └── README.md
├── 05-storage-lvm/
│   └── README.md
├── 06-systemd/
│   └── README.md
├── 07-networking/
│   └── README.md
├── 08-firewalld/
│   └── README.md
├── 09-selinux/
│   └── README.md
├── 10-processes-scheduling/
│   └── README.md
├── 11-troubleshooting-recovery/
│   └── README.md
└── 12-backup-recovery/
    └── README.md
```

Module folders are added as each module is completed.

---

## How Each Module Is Documented

Every module `README.md` follows the same structure:

1. **Objectives:** what the module teaches
2. **Concepts:** simple explanations with enterprise and datacenter context
3. **Commands practiced:** a table of commands and their purpose
4. **Terminal output:** real output captured from the VM
5. **Verification:** how I confirmed the result
6. **Troubleshooting notes:** errors encountered and fixes
7. **Key takeaways:** the main lessons in my own words
8. **Self-check questions:** scenario and interview-style questions with my answers

---

## Conventions

- Commands run as a **normal user** are shown with `$`; commands run as **root** are shown with `#`.
- `sudo` is used wherever privilege is required.
- Terminal output is real, copied from my own VM.
- Hostnames, IP addresses and any sensitive values are lab-only and not production data.
- Folder names are lowercase, hyphen-separated and numbered in learning order.

---

## References

- [Red Hat Enterprise Linux 10 Documentation](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10)
- [Red Hat Developer Program](https://developers.redhat.com/)
- Built-in manuals on the system: `man <command>`, `<command> --help`, `info <command>`

---

## Author

**Emdadul Haque Talha**
BSc in Computer Science and Engineering | Intern Engineer, System/Server Team, Express Systems Limited

- GitHub: https://github.com/ehtalha
- LinkedIn: https://www.linkedin.com/in/emdadul-haque-talha-606939276?utm_source=share_via&utm_content=profile&utm_medium=member_android

---

## Disclaimer

This repository documents personal learning in an isolated lab environment. Commands should be tested in a lab or non-production system before use on real servers. Always follow your organization's change-management and security policies.
