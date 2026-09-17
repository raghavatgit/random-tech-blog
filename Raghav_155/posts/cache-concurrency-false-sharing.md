# Cache Line Contention and False Sharing in Multithreaded Systems

## The L1/L2 Cache Hierarchy
Modern x86-64 and ARM CPUs organize cache memory into discrete 64-byte chunks known as **cache lines**.
When a thread accesses a 4-byte integer in memory, the CPU loads the entire 64-byte line into its L1/L2 data cache.

## What is False Sharing?
False sharing occurs when two independent threads running on separate CPU cores modify distinct variables that reside within the same 64-byte cache line:
- Core 1 writes to `struct.counterA`.
- The MESI cache coherency protocol invalidates the entire 64-byte cache line in Core 2's cache.
- Core 2 attempts to write to `struct.counterB` and experiences an L1 cache miss, forcing a cache line reload over the CPU interconnect.
- Although there is no program-level data race or logical lock contention, performance degrades drastically due to cache line bouncing.

## Mitigation Strategies

### 1. Structure Alignment and Padding
In C++11 and modern C:
```cpp
struct alignas(64) WorkerStats {
    uint64_t counterA;
    char padding[56]; // Explicitly pads out to 64-byte cache line boundary
};
```

### 2. Rust CachePadded
In Rust concurrency (e.g. `crossbeam-utils`):
```rust
use crossbeam_utils::CachePadded;

struct SharedState {
    counter_a: CachePadded<AtomicU64>,
    counter_b: CachePadded<AtomicU64>,
}
```
`CachePadded` aligns and pads the atomic primitives to 64 or 128 bytes depending on target CPU architecture, isolating cache lines and preventing interconnect saturation.
