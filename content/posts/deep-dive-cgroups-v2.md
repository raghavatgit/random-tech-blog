---
title: "Deep Dive into Linux cgroups v2: The Modern Container Primitive"
date: 2026-09-25T10:00:00+05:30
tags: ["linux", "kernel", "containers", "systems"]
---

# Deep Dive into Linux cgroups v2: The Modern Container Primitive

Control Groups (cgroups) are the foundational Linux kernel feature providing resource metering, limits, and isolation for processes. While cgroups v1 powered the initial container revolution (Docker, LXC), its orthogonal, uncoordinated hierarchies introduced deep architectural flaws:
- Writeback pages could not be properly attributed to the block I/O controller because the memory and blkio cgroups were completely disjoint hierarchies.
- Multiple competing trees complicated OOM handling.

## The Unified Hierarchy of cgroups v2
Merged upstream in Linux 4.5 and adopted as default in modern distributions, cgroups v2 enforces a single unified hierarchy rooted at `/sys/fs/cgroup`.

### Key Controllers
1. **Memory (`memory.max` & `memory.high`)**:
   - `memory.high` acts as a soft throttle boundary. When a workload crosses this threshold, the kernel slows down memory allocations and initiates proactive reclaim without invoking the aggressive OOM killer.
2. **CPU (`cpu.max`)**:
   - Expressed as a bandwidth pair `[quota, period]`, allowing sub-millisecond precision over CFS core scheduling.
3. **PSI (Pressure Stall Information)**:
   - Quantifies CPU, memory, and I/O resource starvation as a percentage of lost execution wall-clock time.
