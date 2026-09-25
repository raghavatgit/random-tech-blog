---
title: "Deep-Dive into jemalloc: Arenas, Thread Caches, and Decay Purging"
date: "2026-09-25"
category: "Systems & Memory"
summary: "An architectural exploration of how jemalloc eliminates lock contention in multithreaded databases."
---

# Deep-Dive into jemalloc: Arenas, Thread Caches, and Decay Purging

In high-concurrency systems (such as Redis, TiDB, or ClickHouse), standard `malloc` implementations can quickly become the primary performance bottleneck due to lock contention on the global heap.

## Sharding Memory: Arenas and tcaches

`jemalloc` tackles heap contention with two primary abstractions:

### 1. Thread Caches (`tcache`)
Each thread maintains its own thread-local cache for allocations under 14 KB. Because only the owning thread ever reads or writes to its `tcache`, small object allocations execute in zero clock cycles of synchronization overhead.

### 2. Arenas
When `tcache` exhausts its quota, it requests memory from an **Arena**. The system instantiates `4 * N_CPUS` arenas, assigning threads across them to ensure that lock contention on arena metadata remains negligible.

## Decay-Based Purging

Rather than abruptly running `madvise(MADV_DONTNEED)` and stalling client threads, `jemalloc` uses smooth time-decay purging (`dirty_decay_ms`). Pages that transition to dirty are held for a designated time window before being returned to the kernel, drastically reducing page fault frequency under bursty workloads.
