# Database Concurrency: ANSI SQL Isolation Levels vs Snapshot Isolation

## ANSI SQL-92 Definitions
The ANSI SQL standard defines 4 isolation levels based on prohibited phenomena:
1. **Read Uncommitted**: Dirty reads permitted.
2. **Read Committed**: Prevents dirty reads; non-repeatable reads permitted.
3. **Repeatable Read**: Prevents non-repeatable reads; phantom reads permitted.
4. **Serializable**: Strictly prevents all concurrency anomalies.

## The Critiques of ANSI Definitions (Berenson et al., 1995)
The ANSI standard defined anomalies strictly in terms of lock-based implementations:
- Ignored multi-version concurrency control (MVCC).
- Left ambiguous anomalies such as **Write Skew** and **Read-Only Transaction Anomalies**.

## Snapshot Isolation (SI)
In MVCC engines (PostgreSQL, InnoDB, CockroachDB):
- Every transaction reads from an immutable snapshot corresponding to the logical timestamp of its start time.
- Readers never block writers; writers never block readers.
- **Write Skew Anomaly**:
  - Suppose constraint: $A + B \ge 0$.
  - Transaction 1 checks $A + B \ge 0$ and subtracts 100 from $A$.
  - Transaction 2 checks $A + B \ge 0$ and subtracts 100 from $B$.
  - Both commit successfully under Snapshot Isolation because their write sets do not overlap, yet $A + B$ becomes negative, violating the invariant!

## Serializable Snapshot Isolation (SSI)
PostgreSQL's true `SERIALIZABLE` uses SSI:
- Tracks read-write dependency locks (SIREAD locks) in memory.
- Detects cycles of `rw-antidependency` edges in the serialization graph (precedence graph).
- Aborts one transaction with a serialization failure (`40001`), delivering true serializability without coarse-grained table locks.
