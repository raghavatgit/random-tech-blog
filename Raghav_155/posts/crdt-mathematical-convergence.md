# Mathematical Foundations of State-Based CRDTs (CvRDT)

## The Join-Semilattice Invariant
In distributed state-based Conflict-Free Replicated Data Types (CvRDT), states form a bounded join-semilattice:
- A partially ordered set $(S, \le)$ equipped with a binary join operator $\sqcup$.
- For any elements $x, y, z \in S$:
  1. $x \le x \sqcup y$ and $y \le x \sqcup y$ (upper bound).
  2. If $x \le z$ and $y \le z$, then $x \sqcup y \le z$ (least upper bound).

## Idempotence, Commutativity, and Associativity
The join operator satisfies:
- **Commutativity**: $x \sqcup y = y \sqcup x$ (arrival order across network is irrelevant).
- **Associativity**: $(x \sqcup y) \sqcup z = x \sqcup (y \sqcup z)$ (message batching structure is irrelevant).
- **Idempotence**: $x \sqcup x = x$ (duplicate network packet delivery causes zero corruption).

## Application: LWW-Element-Set
An Element Set storing members $e$ with timestamp $t$:
- $\text{AddSet}: \{ (e, t_{\text{add}}) \}$
- $\text{RemoveSet}: \{ (e, t_{\text{remove}}) \}$
- Element $e$ is present in the set if $(e, t_{\text{add}}) \in \text{AddSet}$ and either:
  1. No entry exists in $\text{RemoveSet}$ for $e$, or
  2. $t_{\text{add}} > t_{\text{remove}}$.
Delivers deterministic causal consistency across unreliable mesh topologies.
