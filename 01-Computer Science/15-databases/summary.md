---
---

## ACID
- **Atomicity**: All-or-nothing. Entire tx succeeds or full rollback.
- **Consistency**: Valid state → valid state. Constraints, FKs, triggers held.
- **Isolation**: Concurrent tx don't interfere. Level governs visibility.
- **Durability**: Committed = permanent. WAL ensures crash survival.

## Normalization
- **1NF**: Atomic columns, no repeating groups, unique rows.
- **2NF**: 1NF + no partial dependency (non-key → whole PK only).
- **3NF**: 2NF + no transitive dependency (non-key → non-key forbidden).
- **BCNF**: 3NF + every determinant is a candidate key.

## SQL vs NoSQL
| Feature | SQL | NoSQL |
|---------|-----|-------|
| Schema | Rigid, predefined | Dynamic, flexible |
| Relations | JOINs | Embedded / references |
| Transactions | ACID | BASE (typically) |
| Scaling | Vertical | Horizontal |
| Examples | PostgreSQL, MySQL | MongoDB, Redis, Cassandra |

## SQL Language Subsets
| Subset | Purpose | Commands |
|--------|---------|----------|
| DDL | Schema | CREATE, ALTER, DROP, TRUNCATE |
| DML | Data | INSERT, UPDATE, DELETE, MERGE |
| DQL | Query | SELECT |
| DCL | Access | GRANT, REVOKE |

## ER Model
| Cardinality | Implementation |
|-------------|----------------|
| 1:1 | FK in either table |
| 1:N | FK on "many" side |
| N:M | Junction table (FKs to both) |

## Locking
| Mechanism | Description |
|-----------|-------------|
| Shared (S) | Read lock. Multiple holders. |
| Exclusive (X) | Write lock. Single holder. |
| Pessimistic | Lock data BEFORE op. |
| Optimistic | Version check AT commit. Retry on conflict. |
| MVCC | Snapshot reads. Readers don't block writers. |
| Deadlock | Txn A waits B, B waits A → DB kills one. |

## Indexes
| Type | Best for | Notes |
|------|----------|-------|
| B-Tree | `=`, `<`, `>`, ranges | Default. Balanced. |
| Hash | `=` only | No range support. |
| Bitmap | Low-cardinality cols | Data warehouses. |
| Clustered | PK lookups, range scans | 1 per table. Data IS index. |
| Non-Clustered | Secondary lookups | N per table. Pointer to row. |
| Composite | Multi-col `(A,B)` | Leftmost prefix rule. |
| Covering | Contains all query cols | Skips row read entirely. |

## Isolation Levels
| Level | Dirty Read | Non-Repeatable | Phantom |
|-------|-----------|---------------|---------|
| Read Uncommitted | ✓ | ✓ | ✓ |
| Read Committed | ✗ | ✓ | ✓ |
| Repeatable Read | ✗ | ✗ | ✓ |
| Serializable | ✗ | ✗ | ✗ |

Defaults: PostgreSQL → Read Committed. MySQL → Repeatable Read.

## Views
- **Standard**: Stored query, computed on-the-fly. Always fresh.
- **Materialized**: Result cached on disk. Fast reads, stale. Needs refresh.

## Stored Procedures
- Code in DB (PL/SQL, T-SQL). Pros: perf, security, 1 network call.
- Cons: hard to debug, vendor lock-in, DB CPU harder to scale.
- **Function**: returns value, usable in `SELECT`. **Procedure**: `CALL`, manages tx.

## CAP & PACELC
| Theorem | Rule |
|---------|------|
| CAP | Pick 2 of {C, A, P}. P inevitable → CP (HBase, Mongo) or AP (Cassandra, Dynamo). |
| PACELC | Partition: A vs C. Else (normal): L vs C. |
| BASE | Basically Available, Soft state, Eventual consistency. |

## Federation · Replication · Sharding
| Pattern | What | Why |
|---------|------|-----|
| Federation | Virtual DB over multiple sources | Query in-place, no ETL |
| Replication | Copy data across nodes | Redundancy + read scale |
| Sharding | Split data across nodes | Write scale + storage |

- **Replication**: Leader-Follower (common), Multi-Leader, Leaderless. Sync vs Async.
- **Sharding**: `hash(key) % N` → shard. Consistent hashing for rebalance. Cross-shard joins = $$.
- **Replication Lag**: delay between leader write and follower visibility.
