---
title: "Demystifying Linux io_uring for Asynchronous IO"
date: 2026-09-24T23:00:00+05:30
tags: ["linux", "kernel", "systems", "performance"]
---

# Demystifying Linux io_uring for Asynchronous I/O

For over two decades, high-performance networking on Linux has relied on `epoll`. While `epoll` revolutionized socket scalability, it came with two fundamental shortcomings:
1. It does not support asynchronous operations on regular disk files.
2. Every notification still requires crossing the user-kernel boundary via `epoll_wait`, followed by another system call (`read`, `write`, `recvmsg`).

In Linux 5.1, Jens Axboe introduced `io_uring`, fundamentally redesigning the Linux I/O model around shared memory lock-free ring buffers.

## The Dual Ring Buffer Architecture
Unlike synchronous system calls, `io_uring` uses two single-producer single-consumer circular buffers mapped directly into both userspace and kernel address spaces:
- **Submission Queue (SQ)**: Userspace pushes Submission Queue Entries (`struct io_uring_sqe`) describing operations.
- **Completion Queue (CQ)**: Kernel processes requests and writes Completion Queue Entries (`struct io_uring_cqe`) containing status and return codes.

## The SQPOLL Revolution
By launching with `IORING_SETUP_SQPOLL`, the kernel dedicates a kernel thread (`io_uring-sq`) to poll the Submission Queue continuously.
Applications can submit thousands of asynchronous requests without executing a single `enter` syscall, eliminating CPU context switch overhead entirely.
