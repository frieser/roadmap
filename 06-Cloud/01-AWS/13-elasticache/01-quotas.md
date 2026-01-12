---
tags: ['cloud', 'roadmap', 'aws', 'elasticache']
---

# ElastiCache Quotas and Engine Differences (Redis vs Memcached)

## Summary
Amazon ElastiCache provides two primary engines: **Redis OSS (and Community Edition)** and **Memcached**. Choosing the right engine depends on requirements for data structures, persistence, and scaling. Understanding service quotas (e.g., the 500-node limit per region) and cluster configurations (Cluster Mode Enabled vs. Disabled) is critical for architecting high-performance caching layers.

## Detailed Explanation

### 1. Redis OSS/CE vs. Memcached: Key Differences

| Feature | Redis OSS/CE | Memcached |
| :--- | :--- | :--- |
| **Data Types** | Advanced (Lists, Sets, Hashes, Geospatial, etc.) | Simple (Strings only) |
| **Persistence** | Yes (Snapshots/RDB and AOF) | No (Purely in-memory) |
| **High Availability** | Yes (Replication groups, Multi-AZ with failover) | No (If node fails, data is lost) |
| **Scaling** | Vertical & Horizontal (Sharding) | Vertical & Horizontal (Adding nodes) |
| **Threading** | Primarily single-threaded (IO threads in 6.0+) | Multi-threaded (Scales with cores) |
| **Transactions** | Yes | No |
| **Pub/Sub** | Yes | No |
| **Max Item Size** | 512 MB | 1 MB |

**Note on Licensing (2025/2026):** Redis 8.0+ uses the AGPLv3 license. For fully open-source requirements, AWS supports **Valkey** as a compatible alternative to the older Redis OSS versions.

### 2. ElastiCache for Redis Cluster Modes

- **Cluster Mode Disabled**:
    - Supports 1 Primary and up to 5 Read Replicas.
    - Total limit of 6 nodes per replication group.
    - Scaling is limited to adding replicas or increasing instance size.
- **Cluster Mode Enabled**:
    - Supports up to **500 shards**.
    - Each shard can have 1 primary and up to 5 replicas.
    - Horizontal scaling (resharding) is possible while the cluster is online.

### 3. Service Quotas (Default Limits)

- **Nodes per Region**: 500 (Standard soft limit, increaseable).
- **Clusters per Region**: 500 (Redis).
- **Memcached Nodes per Cluster**: 40.
- **Nodes per Replication Group (Redis)**:
    - Cluster Mode Disabled: 6.
    - Cluster Mode Enabled: 500 (shards).
- **Reserved Memory**: Controlled by `reserved-memory-percent` (default ~25%) to ensure enough memory for maintenance/failover.

## Interview Questions

**Q1: When should you choose Memcached over Redis for an ElastiCache implementation?**
**A:** Choose Memcached if you need a simple, high-performance, multi-threaded cache that doesn't require persistence, advanced data structures, or replication. It is ideal for "flat" caching of objects up to 1MB where the application handles data partitioning.

**Q2: What is the maximum number of replicas allowed per Redis primary node?**
**A:** In ElastiCache for Redis, you can have up to **5 read replicas** per primary node (total 6 nodes in the replication group).

**Q3: Explain the difference between horizontal scaling in Memcached vs. Redis Cluster.**
**A:** In Memcached, horizontal scaling involves adding more nodes to the cluster, and the client (using consistent hashing) decides where to store data. In Redis Cluster Mode Enabled, horizontal scaling involves adding shards and **resharding** (moving hash slots) between old and new shards, often managed by ElastiCache while the cluster stays online.

**Q4: What is the default node limit per region in AWS ElastiCache, and can it be increased?**
**A:** The default limit is **500 nodes per region**. This is a soft limit and can be increased by submitting a service quota increase request to AWS Support.

**Q5: How does ElastiCache handle Redis persistence, and does it affect performance?**
**A:** ElastiCache supports RDB snapshots and AOF. RDB takes point-in-time backups, while AOF logs every write. Both can impact performance (especially AOF), and it is recommended to set a `reserved-memory` buffer to prevent the node from running out of RAM during the fork process required for snapshots.
