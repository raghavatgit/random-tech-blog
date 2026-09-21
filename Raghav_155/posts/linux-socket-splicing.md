# Zero-Copy Linux Socket Splicing: Moving Packets Without Context Switches

## The Kernel Page Cache Hop Problem
In traditional proxy and gateway architectures (HAProxy, Envoy, Nginx):
1. Packets arrive on ingress socket `fd_in`.
2. Kernel executes `recv(2)` copying payload from kernel socket buffer into user space buffer.
3. Proxy inspects headers, identifies backend server, and calls `send(2)` on egress socket `fd_out`.
4. Kernel copies payload from user space into egress kernel socket buffer.
5. Incurs 4 system calls, 4 context switches, and 2 full-memory copies per network packet.

## High-Throughput Socket Splicing with splice(2)
Linux `splice(2)` establishes a direct pipeline between two sockets using an intermediate anonymous pipe:
```c
int pipefd[2];
pipe(pipefd);

// Direct socket-to-socket transfer
while ((bytes = splice(fd_in, NULL, pipefd[1], NULL, 65536, SPLICE_F_MOVE | SPLICE_F_NONBLOCK)) > 0) {
    splice(pipefd[0], NULL, fd_out, NULL, bytes, SPLICE_F_MOVE | SPLICE_F_NONBLOCK);
}
```
Page descriptors are passed directly between socket buffers without touching user space, reducing CPU cache pollution and achieving line-rate 40Gbps+ forwarding throughput.
