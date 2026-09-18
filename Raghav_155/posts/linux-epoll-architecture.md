# Linux epoll Architecture: Readiness Models and the Thundering Herd

## From select(2) and poll(2) to epoll(7)
Early UNIX multiplexing required linear scans:
- `select(2)` and `poll(2)` pass the entire file descriptor set from user space to kernel space on every invocation ($O(N)$ memory copies).
- Kernel must iterate through all $N$ file descriptors to check poll status, even if only 1 socket received data.

## The epoll In-Kernel Data Structures
`epoll(7)` splits state management into distinct system calls (`epoll_create`, `epoll_ctl`, `epoll_wait`):
1. **Red-Black Tree**: Stores monitored file descriptors. Insertions, lookups, and deletions execute in $O(\log N)$ time.
2. **Ready List (Doubly Linked List)**: When an I/O event occurs on an FD, the hardware interrupt handler places a pointer into the ready list via the socket wait queue callback.
3. `epoll_wait(2)` only examines the ready list in $O(1)$ time, copying only active events back to user space.

## Edge-Triggered (ET) vs Level-Triggered (LT)
- **Level-Triggered (Default)**: `epoll_wait` continuously returns the FD as long as data remains in the receive buffer. Forgiving of partial reads.
- **Edge-Triggered (EPOLLET)**: `epoll_wait` reports the event only when readiness transitions from unready to ready. Applications must drain the socket completely in a non-blocking loop until `EAGAIN` or `EWOULDBLOCK`.

## The Thundering Herd Problem and EPOLLEXCLUSIVE
When multiple worker processes listen on the same server socket:
- New connection wake-ups historically awakened all sleeping workers, triggering massive context-switch overhead.
- Linux 4.5 introduced `EPOLLEXCLUSIVE`: kernel wakes exactly one worker per incoming connection, eliminating thundering herd contention.
