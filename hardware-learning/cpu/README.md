# CPU (Central Processing Unit)

## 1. Introduction

The Central Processing Unit (CPU) is one of the most important components of a computer system. It executes instructions, processes data, and performs calculations required by the operating system and applications.

In enterprise environments, CPUs are used in physical servers, virtual machines, databases, application servers, and cloud infrastructure.

This document covers the basic concepts of CPU architecture, important CPU components, enterprise use cases, and practical Linux commands used to monitor CPU information in Red Hat Enterprise Linux (RHEL).

## 2. Learning Objectives

After completing this topic, I can:

- Explain the basic function of a CPU.
- Understand CPU sockets, cores, threads, and cache memory.
- Understand the difference between physical CPUs and virtual CPUs (vCPUs).
- Explain the basic role of a CPU in server virtualization.
- Identify CPU information using Linux commands.
- Monitor CPU utilization in RHEL.
- Perform basic CPU-related practical exercises in a virtual machine.

## 3. Basic CPU Concepts

### 3.1 What Is a CPU?

A CPU is the main processing component of a computer. It reads and executes instructions and performs operations required by software.

In a server, the CPU processes workloads from operating systems, applications, databases, and virtual machines.

### 3.2 CPU Socket

A CPU socket is the physical connection between a processor and the motherboard.

A physical server may have one or more CPU sockets, depending on its hardware design.

### 3.3 CPU Core

A core is an individual processing unit inside a CPU.

A CPU with multiple cores can execute multiple tasks at the same time.

### 3.4 CPU Thread

A thread is a sequence of instructions that can be scheduled for execution by the CPU.

Some processors support Simultaneous Multithreading (SMT), which allows one physical core to handle multiple hardware threads.

Intel commonly calls this technology Hyper-Threading.

### 3.5 CPU Cache

CPU cache is a small, high-speed memory located inside or close to the processor.

It stores frequently used data and instructions to reduce access time.

Common cache levels:

- **L1 Cache:** Smallest and fastest cache.
- **L2 Cache:** Larger than L1 but slower.
- **L3 Cache:** Usually larger and shared among multiple CPU cores.

### 3.6 Clock Speed

Clock speed represents the operating frequency of a CPU. It is commonly measured in Gigahertz (GHz).

Clock speed alone does not determine overall CPU performance. CPU architecture, core count, cache, and workload also affect performance.

## 4. CPU Architecture Overview

A simplified processing flow:

1. The CPU receives instructions from a running program.
2. It fetches the required instructions and data.
3. It decodes the instructions.
4. It executes the required operations.
5. It stores or returns the results.

The CPU works together with RAM, storage, motherboard, and the operating system to complete these operations.

## 5. CPU in Enterprise Server Infrastructure

### 5.1 Physical CPU

A physical CPU is installed in a server motherboard.

Its resources include:
- Physical sockets
- Physical cores
- Hardware threads
- CPU cache

### 5.2 Virtual CPU (vCPU)

A vCPU is a virtual processing resource assigned to a virtual machine.

A hypervisor, such as VMware ESXi, schedules virtual machine workloads onto the physical CPU resources of the host.

### 5.3 CPU in Virtualization

In a virtualized environment:

- The physical server provides CPU resources.
- The hypervisor manages and schedules those resources.
- Virtual machines receive allocated vCPUs.
- Guest operating systems and applications use those vCPUs to execute workloads.

CPU allocation should be planned based on workload requirements and available physical resources.

## 6. Practical Lab Environment

| Component | Details |
|---|---|
| Hypervisor | VMware Workstation Pro |
| Operating System | Red Hat Enterprise Linux (RHEL) |
| Environment | Virtual Machine |
| User | Root / Privileged Linux User |
| Topic | CPU Information and Monitoring |

## 7. Linux Commands for CPU Information

### 7.1 Display CPU Architecture and Information

```bash
lscpu
```

**Purpose:** Displays CPU architecture, CPU count, sockets, cores, threads, cache, and other CPU-related information.

### 7.2 Display the Number of Available Processing Units

```bash
nproc
```

**Purpose:** Displays the number of processing units available to the current process.

### 7.3 Display Detailed CPU Information

```bash
cat /proc/cpuinfo
```

**Purpose:** Displays detailed CPU information provided by the Linux kernel.

### 7.4 Display CPU Information for Each Processor

```bash
grep "processor" /proc/cpuinfo
```

**Purpose:** Displays processor index entries from `/proc/cpuinfo`.

### 7.5 Display CPU Model Name

```bash
grep "model name" /proc/cpuinfo
```

**Purpose:** Displays the processor model name.

### 7.6 Display the Number of CPU Cores

```bash
grep "cpu cores" /proc/cpuinfo
```

**Purpose:** Displays the physical core count reported for each CPU package.

### 7.7 Display CPU Socket, Core, and Thread Information

```bash
lscpu | grep -E 'CPU\(s\)|Core|Thread|Socket'
```

**Purpose:** Filters important CPU topology information from the `lscpu` output.

### 7.8 Display System Uptime and Load Average

```bash
uptime
```

**Purpose:** Displays system uptime and load averages.

Load average represents the average number of tasks that are runnable or waiting for certain resources. It is not the same as CPU utilization percentage.

### 7.9 Monitor CPU and Process Usage

```bash
top
```

**Purpose:** Provides real-time information about running processes, CPU utilization, memory usage, and system activity.

Press `q` to exit.

### 7.10 Monitor System Performance Statistics

```bash
vmstat
```

**Purpose:** Displays system performance information, including processes, memory, CPU, and system activity.

### 7.11 Monitor Performance at One-Second Intervals

```bash
vmstat 1
```

**Purpose:** Updates performance statistics every one second.

Press `Ctrl + C` to stop monitoring.

## 8. Practical CPU Load Testing

### 8.1 Generate CPU Workload

```bash
yes > /dev/null
```

**Purpose:** Generates continuous processing activity by repeatedly producing output and redirecting it to `/dev/null`.

This command can increase CPU utilization.

**Important:** Stop the process after the test by pressing:

```text
Ctrl + C
```

Do not run this command unnecessarily on production servers.

### 8.2 Observe CPU Utilization

Open another terminal and execute:

```bash
top
```

Observe the CPU utilization and the process running the `yes` command.

### 8.3 Monitor Performance Using vmstat

```bash
vmstat 1
```

Observe changes in CPU-related columns:

- `us`: CPU time spent running user processes.
- `sy`: CPU time spent running system processes.
- `id`: CPU idle time.
- `wa`: CPU time waiting for I/O.
- `st`: CPU time stolen by the hypervisor from a virtual machine.

These values help understand CPU activity in a virtualized environment.

## 9. Practical Lab Results

Record the actual output from the RHEL virtual machine.

| Command | Observation |
|---|---|
| `lscpu` | CPU architecture, sockets, cores, and threads |
| `nproc` | Available processing units |
| `cat /proc/cpuinfo` | Detailed CPU information |
| `top` | Real-time CPU and process utilization |
| `uptime` | System uptime and load average |
| `vmstat 1` | CPU and system performance statistics |
| `yes > /dev/null` | CPU workload generation and monitoring |

**Note:** CPU values may differ between VMware virtual machines and physical servers because the hypervisor controls the CPU resources presented to the guest operating system.

## 10. Key Takeaways

- CPU is responsible for executing instructions and processing data.
- A CPU can contain multiple cores and hardware threads.
- CPU sockets connect processors to the server motherboard.
- Cache memory helps reduce data access latency.
- Clock speed is one factor in CPU performance.
- Physical CPUs provide processing resources to virtualized environments.
- vCPUs are virtual processing resources assigned to virtual machines.
- Linux provides several commands to inspect and monitor CPU information.
- CPU monitoring is important for performance analysis and troubleshooting.

## 11. Conclusion

This topic provided an introduction to CPU fundamentals, architecture, and its role in enterprise server infrastructure.

Through practical exercises in RHEL running on VMware Workstation Pro, I learned how to identify CPU resources, inspect processor information, monitor system performance, and generate a basic CPU workload.

This knowledge provides a foundation for further study of server hardware, virtualization, and enterprise infrastructure.

## 12. References

- [Red Hat Enterprise Linux Documentation](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/)
- [Linux Manual Pages](https://man7.org/linux/man-pages/)
- [Linux `lscpu` Manual](https://man7.org/linux/man-pages/man1/lscpu.1.html)
- [Linux `top` Manual](https://man7.org/linux/man-pages/man1/top.1.html)
- [Linux `vmstat` Manual](https://man7.org/linux/man-pages/man8/vmstat.8.html)
