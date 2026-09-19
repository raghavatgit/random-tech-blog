# Inside the Linux CFS Scheduler: Weights, Slices, and Latency Targets

## Priority to Weight Mapping
The Linux Completely Fair Scheduler assigns nice values [-20, 19] to logarithmic weights:
```c
const int sched_prio_to_weight[40] = {
    88761, 71755, 56483, ... // nice -20 to -1
    1024,                     // nice 0 (default)
    820, 655, ...            // nice +1 to +19
};
```
Every step of nice level corresponds to approximately a 10% change in relative CPU execution proportion.

## Calculation of Physical Timeslice
Given a target scheduling period $T_{\text{latency}}$ and total runqueue weight $W = \sum w_i$:
$$\text{slice}_i = T_{\text{latency}} \times \frac{w_i}{W}$$

## Virtual Runtime Increment Formula
When task $i$ runs for physical duration $\Delta t$:
$$\Delta \text{vruntime}_i = \Delta t \times \frac{w_0}{w_i}$$
where $w_0 = 1024$ (nice 0 weight).
- If task priority is high ($w_i > 1024$), $\Delta \text{vruntime} < \Delta t$.
- It advances slowly in the red-black tree, remaining towards the left edge to receive more CPU timeslices.
