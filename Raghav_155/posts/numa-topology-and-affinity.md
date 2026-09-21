# NUMA Node Interconnects and Thread Affinity in Low-Latency Engines

## Cache Coherency Traffic Over Interconnects
In multi-socket architectures:
- Each CPU package communicates over high-speed point-to-point links (Intel Ultra Path Interconnect - UPI, AMD Infinity Fabric).
- When a thread on Socket 0 reads memory allocated on Socket 1's physical memory controller:
  - Traverses the inter-socket interconnect.
  - Sits on UPI bus arbitration queues, adding $\approx 60-100\text{ns}$ latency.
  - Squeezes memory bandwidth for other cores on Socket 0.

## Thread Pinning and Memory Locality
To achieve deterministic p99.99 latency in trading systems and database engines:
```bash
# Launch process pinned strictly to NUMA node 0 CPU cores and memory
numactl --cpunodebind=0 --membind=0 ./high_throughput_engine
```
Guarantees memory allocations, L3 cache accesses, and I/O DMA buffers originate strictly from local PCI-e root complexes and memory buses.
