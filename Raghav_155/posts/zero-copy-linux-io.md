# Zero-Copy I/O in Linux: sendfile, splice, and io_uring

## The Traditional Read/Write Overhead
Transferring a file from disk to a network socket using standard `read(2)` and `write(2)` incurs 4 context switches and 4 data copies:
1. **Disk to Page Cache**: OS performs DMA transfer from disk to kernel page cache.
2. **Page Cache to User Buffer**: Kernel copies data into user application memory via CPU.
3. **User Buffer to Socket Buffer**: Kernel copies data from user space to kernel socket buffer.
4. **Socket Buffer to NIC**: NIC performs DMA transfer from socket buffer to network card.

During this flow, data crosses user-space boundaries twice without modification.

## sendfile(2) System Call
Introduced to eliminate user-space hops:
```c
#include <sys/sendfile.h>
ssize_t sendfile(int out_fd, int in_fd, off_t *offset, size_t count);
```
With scatter-gather DMA support in modern NICs:
- Data moves directly from Kernel Page Cache to NIC DMA ring.
- Socket buffer receives only file descriptor descriptors (headers and pointers).
- CPU copy overhead drops to zero. Context switches reduce to 2.

## splice(2) and Linux Pipes
`splice(2)` moves data between arbitrary file descriptors using kernel pipe buffers without copying to user space.

## io_uring Zero-Copy Transmission
Linux 5.19+ introduced `IORING_OP_SEND_ZC`:
- Eliminates page reference counting overhead and socket lock contention.
- Enables high-throughput multi-gigabit streaming servers with deterministic latency profiles.
