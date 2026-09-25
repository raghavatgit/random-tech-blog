---
title: "Understanding Linux VFS: Dentry Caches and Lockless RCU-Walk Mechanics"
date: "2026-09-25"
category: "Systems"
summary: "How modern Linux kernels achieve linear path resolution scaling across hundreds of CPU cores using lockless RCU path walks."
---

# Understanding Linux VFS: Dentry Caches and Lockless RCU-Walk Mechanics

When an application invokes `open("/etc/hosts", O_RDONLY)`, the Linux kernel does not immediately read raw disk sectors. Instead, it queries the **Virtual File System (VFS)** layer, specifically the **dentry cache** (`dcache`).

## The Multi-Core Path Lookup Bottleneck

In early versions of Linux, resolving a path component required:
1. Locking the parent directory inode.
2. Looking up the child in the hash bucket.
3. Incrementing reference counters (`dget`).
4. Releasing the lock.

On a 64-core or 128-core NUMA system running microservices, hundreds of threads reading shared configuration files simultaneously induced severe cache-line bouncing. The CPU spent more cycles arbitrating bus locks than parsing filenames.

## Enter RCU-Walk

Linux introduced **RCU-walk** in `fs/namei.c`. Path traversal is executed inside an `rcu_read_lock()` block:
- **Zero Reference Counting**: Intermediate dentries are traversed without modifying their atomic reference counts.
- **Sequence Locks (`d_seq`)**: Each dentry contains a sequence counter. If a directory is concurrently renamed or unlinked, the sequence number changes, signaling to the reader that the traversed path may be invalid.
- **Graceful Fallback**: If an invalidation or blocking operation occurs, the kernel seamlessly rolls back and falls back to traditional ref-walk.

This optimization ensures that static file lookups in high-concurrency web servers remain purely read-only down to the hardware cache lines.
