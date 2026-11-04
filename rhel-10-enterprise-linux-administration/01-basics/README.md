# Module 1: RHEL Basics

| | |
|---|---|
| **Environment** | Red Hat Enterprise Linux 10.2 on VMware Workstation |
| **Author** | Emdadul Haque Talha |
| **Date completed** | `November 04,2026` |
| **Status** | `Completed` |

---

## Table of Contents

1. [Objectives](#objectives)
2. [Lesson 1.1: System Identity](#lesson-11-system-identity)
3. [Lesson 1.2: Navigating the Filesystem](#lesson-12-navigating-the-filesystem)
4. [Lesson 1.3: File and Directory Operations](#lesson-13-file-and-directory-operations)
5. [Lesson 1.4: File System Hierarchy](#lesson-14-file-system-hierarchy)
6. [Lesson 1.5: Help System and Viewing Files](#lesson-15-help-system-and-viewing-files)
7. [Troubleshooting Notes](#troubleshooting-notes)
8. [Command Cheat Sheet](#command-cheat-sheet)
9. [Key Takeaways](#key-takeaways)
10. [Self-Check Questions](#self-check-questions)
11. [Completion Checklist](#completion-checklist)

---

## Objectives

After completing this module, the learner should be able to:

- Explain what RHEL is and why enterprises use it.
- Identify the operating system version, kernel, hostname and current user of any server.
- Navigate the Linux filesystem using absolute and relative paths.
- Create, copy, move, rename and delete files and directories safely.
- Describe the purpose of the main directories in the File System Hierarchy.
- Use manual pages, `--help`, pipes and basic text tools to find information independently.

---

## Lesson 1.1: System Identity

### Concepts

- **Kernel:** the core of the operating system that manages hardware (CPU, memory, disks, network interfaces).
- **Distribution:** a kernel combined with tools, libraries and a package manager.
- **RHEL:** an enterprise distribution with a support lifecycle of approximately ten years per major release, vendor support from Red Hat, and hardware and software certification.
- **Version format:** `10.2` denotes major release 10, minor release 2.
- **Prompt:** `$` indicates a normal user; `#` indicates the root user.
- **Command syntax:** `command [options] [arguments]`, for example `ls -l /etc`.

### Why It Matters

Before performing any work on a server, an engineer must confirm its operating system version, kernel and identity. Patching, vendor support tickets, compatibility checks and documentation all depend on this information.

### Commands Practiced

| Command | Purpose |
|---|---|
| `whoami` | Display the current username |
| `id` | Display user ID, group ID and group memberships |
| `hostname` | Display the system hostname |
| `hostnamectl` | Display hostname, OS, kernel and virtualization details |
| `cat /etc/os-release` | Display the OS name and version |
| `cat /etc/redhat-release` | Display the Red Hat release string |
| `uname -r` | Display the kernel version |
| `uname -a` | Display all kernel and system information |
| `uptime` | Display time since boot and load average |
| `who` | Display logged-in users |
| `date` | Display the current date and time |
| `lscpu` | Display CPU information |
| `free -h` | Display memory usage in human-readable form |
| `df -h` | Display filesystem usage in human-readable form |

### Terminal Output

```text
$ cat /etc/os-release
<paste output here>

$ uname -r
<paste output here>

$ hostnamectl
<paste output here>
```

### Verification

- `/etc/os-release` shows `NAME="Red Hat Enterprise Linux"` and `VERSION_ID="10.2"`.
- `uname -r` shows a kernel string containing `el10`, which identifies Enterprise Linux 10.

---

## Lesson 1.2: Navigating the Filesystem

### Concepts

- Linux uses a **single directory tree** that begins at the root directory `/`. There are no drive letters.
- **Absolute path:** begins with `/` and is valid from anywhere (for example `/etc/ssh`).
- **Relative path:** is resolved from the current directory (for example `../log`).
- Special symbols: `.` is the current directory, `..` is the parent directory, `~` is the home directory, and `-` is the previous directory.
- Names are **case-sensitive**: `File.txt` and `file.txt` are different files.

### Why It Matters

Servers have no graphical interface. Engineers move between configuration, log and data directories continuously, and a wrong path in a command can affect the wrong data.

### Commands Practiced

| Command | Purpose |
|---|---|
| `pwd` | Print the current working directory |
| `ls` | List directory contents |
| `ls -l` | Long listing (permissions, owner, size, date) |
| `ls -a` | Include hidden files |
| `ls -lah` | Long listing, hidden files, human-readable sizes |
| `ls -R` | Recursive listing |
| `cd /etc` | Change to an absolute path |
| `cd ..` | Move up one level |
| `cd ~` or `cd` | Move to the home directory |
| `cd -` | Return to the previous directory |

### Terminal Output

```text
$ pwd
/home/ghost49

$ ls -lah ~

total 32K
drwx--x---+ 17 ghost49 ghost49 4.0K Sep 13 16:05 .
drwxr-xr-x.  6 root    root      54 Sep 13 15:29 ..
drwxr-xr-x.  3 ghost49 ghost49   51 Sep 13 14:54 AIUB
-rw-------.  1 ghost49 ghost49 2.5K Nov  2 11:48 .bash_history
-rw-r--r--.  1 ghost49 ghost49   18 Oct 29  2024 .bash_logout
-rw-r--r--.  1 ghost49 ghost49  144 Oct 29  2024 .bash_profile
-rw-r--r--.  1 ghost49 ghost49  522 Oct 29  2024 .bashrc
drwx------. 10 ghost49 ghost49 4.0K Sep  9 16:07 .cache
drwx------. 14 ghost49 ghost49 4.0K Sep 13 14:51 .config
drwxr-xr-x.  3 ghost49 ghost49   37 Sep 10 17:53 Desktop
drwxr-xr-x.  2 ghost49 ghost49    6 Sep  7 12:36 Documents
drwxr-xr-x.  2 ghost49 ghost49    6 Sep  7 12:36 Downloads
drwxr-x---.  2 EEE     FE         6 Sep 13 16:05 FE
drwxrwx---+  2 CSE     FST       20 Sep 14 12:22 FST
drwx------.  4 ghost49 ghost49   32 Sep  7 12:36 .local
drwxr-xr-x.  4 ghost49 ghost49   39 Sep  7 12:26 .mozilla
drwxr-xr-x.  2 ghost49 ghost49    6 Sep  7 12:36 Music
drwxr-xr-x.  2 ghost49 ghost49    6 Sep  7 12:36 Pictures
drwxr-xr-x.  2 ghost49 ghost49    6 Sep  7 12:36 Public
drwxr-xr-x.  2 ghost49 ghost49    6 Sep  7 12:36 Templates
drwxr-xr-x.  2 ghost49 ghost49    6 Sep  7 12:36 Videos
-rw-------.  1 ghost49 ghost49 1.3K Sep 13 14:54 .viminfo

$ cd /var/log && pwd
/var/log
```

### Verification

Run `pwd` after each `cd` command to confirm the location.

---

## Lesson 1.3: File and Directory Operations

### Concepts

- Everything in Linux is treated as a file, including directories and devices.
- Operations performed in the shell are **immediate and generally irreversible**. There is no recycle bin.
- Redirection: `>` overwrites a file, `>>` appends to a file.

### Why It Matters

Server work involves creating directories for data, copying configuration files before editing them, and removing old files. Taking a backup copy before modification is a standard professional habit.

### Commands Practiced

| Command | Purpose |
|---|---|
| `mkdir lab1` | Create a directory |
| `mkdir -p lab1/a/b/c` | Create nested directories |
| `touch file1.txt` | Create an empty file or update its timestamp |
| `echo "text" > file.txt` | Write text to a file (overwrite) |
| `echo "more" >> file.txt` | Append text to a file |
| `cp file1.txt file2.txt` | Copy a file |
| `cp -r dir1 dir2` | Copy a directory recursively |
| `mv file2.txt renamed.txt` | Rename or move a file |
| `rm file.txt` | Delete a file |
| `rm -i file.txt` | Delete with confirmation |
| `rmdir emptydir` | Delete an empty directory |
| `rm -r dir` | Delete a directory and its contents |

### Terminal Output

```text
$ mkdir -p ~/lab1/projects/docs
$ touch ~/lab1/projects/docs/notes.txt
$ echo "RHEL practice" > ~/lab1/projects/docs/notes.txt
$ cp ~/lab1/projects/docs/notes.txt ~/lab1/projects/docs/notes.txt.bak
$ ls -l ~/lab1/projects/docs
<paste output here>
```

### Verification

Use `ls -l` and `cat` to confirm that files exist and contain the expected content.

### Safety Note

Never run `rm -rf /` or `rm -rf` with an unverified path or variable on any system. Always confirm the path with `pwd` and `ls` first.

---

## Lesson 1.4: File System Hierarchy

### Concepts

The File System Hierarchy Standard (FHS) defines the purpose of each top-level directory.

| Directory | Purpose |
|---|---|
| `/` | Root of the entire directory tree |
| `/boot` | Kernel, initramfs and bootloader files |
| `/etc` | System-wide configuration files |
| `/home` | Home directories of regular users |
| `/root` | Home directory of the root user |
| `/var` | Variable data: logs, caches, spool, databases |
| `/usr` | Installed programs, libraries and documentation |
| `/bin`, `/sbin`, `/lib`, `/lib64` | Symbolic links to the corresponding directories under `/usr` |
| `/tmp` | Temporary files |
| `/dev` | Device files (disks, terminals) |
| `/proc`, `/sys` | Virtual filesystems exposing kernel and hardware information |
| `/run` | Runtime data created since boot |
| `/mnt`, `/media` | Mount points for temporary and removable filesystems |
| `/opt` | Optional third-party software |
| `/srv` | Data served by the system (web, FTP) |

### Why It Matters

Knowing where things live determines how quickly an engineer can resolve incidents: configuration is in `/etc`, logs are in `/var/log`, and disk space problems frequently originate in `/var` or `/home`. Storage engineers also use this layout to decide which directories to place on separate volumes.

### Commands Practiced

| Command | Purpose |
|---|---|
| `ls -l /` | List top-level directories and symbolic links |
| `ls /etc \| head` | Preview configuration files |
| `ls /var/log` | List log files |
| `cat /proc/cpuinfo \| head` | View CPU data exposed by the kernel |
| `cat /proc/meminfo \| head` | View memory data exposed by the kernel |
| `ls -l /dev \| head` | View device files |
| `du -sh /var /usr /home` | Check directory sizes (use `sudo` if access is denied) |

### Terminal Output

```text
$ ls -l /
<paste output here>

$ du -sh /var /usr /home
<paste output here>
```

### Verification

`ls -l /` should show `bin -> usr/bin` and `sbin -> usr/sbin`, confirming the merged `/usr` layout.

---

## Lesson 1.5: Help System and Viewing Files

### Concepts

- **Manual pages** (`man`) are the authoritative built-in documentation for commands and configuration files.
- **`--help`** gives a short usage summary.
- **Pager (`less`)** displays long output one screen at a time.
- **Pipe (`|`)** sends the output of one command as the input of another.
- **`grep`** filters lines that match a pattern.

### Why It Matters

Production servers frequently have no internet access during an incident. The ability to read local documentation and filter large outputs is what distinguishes an independent engineer.

### Commands Practiced

| Command | Purpose |
|---|---|
| `man ls` | Open the manual page for `ls` |
| `ls --help` | Show a usage summary |
| `whatis ls` | One-line description of a command |
| `man -k <keyword>` | Search manual pages by keyword |
| `cat file` | Print a whole file |
| `less file` | Page through a file |
| `head -n 5 file` | Show the first lines |
| `tail -n 5 file` | Show the last lines |
| `wc -l file` | Count lines |
| `grep root /etc/passwd` | Show lines containing `root` |
| `grep -i pattern file` | Case-insensitive search |
| `command \| grep pattern` | Filter command output |
| `journalctl -n 10 --no-pager` | Show the last ten system log entries |

**Navigation keys in `man` and `less`:** `Space` next page, `b` previous page, `/text` search forward, `n` next match, `q` quit.

### Terminal Output

```text
$ whatis ls
<paste output here>

$ grep root /etc/passwd
<paste output here>

$ ls /etc | wc -l
<paste output here>
```

### Verification

Confirm that `grep` returns only matching lines and that `q` exits `man` and `less` cleanly.

---

## Troubleshooting Notes

| Symptom | Likely Cause | Resolution |
|---|---|---|
| `command not found` | Misspelling, wrong case, or package not installed | Check spelling; use `whatis` or `man -k`; install the package later with DNF |
| `No such file or directory` | Incorrect path | Verify with `pwd` and `ls`; check absolute versus relative path |
| `Permission denied` | Insufficient privileges | Check ownership with `ls -l`; use `sudo` only where appropriate |
| `man` reports no entry or `man -k` returns nothing | Manual database not built | Run `sudo mandb`, then retry |
| Terminal appears frozen | A command is still running or a pager is open | Press `Ctrl+C` to stop a command, or `q` to leave a pager |

Additional notes from my own practice:

- `<error encountered and how it was resolved>`

---

## Command Cheat Sheet

```bash
# Identity
whoami; id; hostnamectl; cat /etc/os-release; uname -r

# Navigation
pwd; ls -lah; cd /path; cd ..; cd ~; cd -

# File operations
mkdir -p a/b; touch f; cp -r src dst; mv old new; rm -i f; rm -r dir

# Viewing and filtering
cat f; less f; head -n 5 f; tail -n 5 f; grep -i word f; cmd | grep word

# Help
man cmd; cmd --help; whatis cmd; man -k keyword
```

---

## Key Takeaways

- `<RHEL version 10.2 means major release 10, minor release 2.>`
- `<The kernel version contains el10, identifying Enterprise Linux 10.>`
- `<The filesystem is one tree starting at /; configuration is in /etc and logs are in /var/log.>`
- `<Deletion in the shell is permanent, so verify paths before removing anything.>`
- `<man pages and --help provide offline documentation.>`

---

## Self-Check Questions

1. **Scenario:** A vendor asks which RHEL release and kernel a server is running. Which commands provide this information, and what exactly would you report?
   - Answer: `<your answer>`

2. **Scenario:** The root filesystem of a server is nearly full. Which top-level directories would you inspect first, and with which command?
   - Answer: `<your answer>`

3. **Scenario:** You must edit `/etc/ssh/sshd_config` on a production server. What do you do before editing, and why?
   - Answer: `<your answer>`

---

## Completion Checklist

- [ ] Confirmed RHEL 10.2 with `/etc/os-release`
- [ ] Navigated using absolute and relative paths
- [ ] Created, copied, moved and deleted files and directories
- [ ] Explored `/`, `/etc`, `/var`, `/usr`, `/proc` and `/dev`
- [ ] Used `man`, `--help`, `less`, `grep` and pipes
- [ ] Answered the self-check questions
- [ ] Committed this README to GitHub

---

**Next module:** [Module 2: Users and Groups](../02-users-groups/)
