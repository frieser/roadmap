---
---

# Document Databases: Senior Backend Overview

Document databases are a type of NoSQL database that store data in flexible, semi-structured documents (usually JSON, BSON, or XML). They are designed for horizontal scalability and developer agility.

## Core Comparison: MongoDB vs. CouchDB vs. RethinkDB

| Feature | MongoDB | CouchDB | RethinkDB |
| :--- | :--- | :--- | :--- |
| **Primary Focus** | Performance & Scalability | Sync & Reliability | Real-time Push |
| **Data Format** | BSON (Binary JSON) | JSON | JSON |
| **Query Language** | Aggregation Pipeline / MQL | MapReduce / Mango | ReQL (Functional) |
| **Consistency Model** | CP (Strong Consistency) | AP (Eventual Consistency) | CP (Strong Consistency) |
| **Transactions** | ACID (since v4.0) | Document-level only | Document-level only |
| **Replication** | Primary-Secondary | Master-Master (Multi-Master) | Primary-Secondary |
| **Best For** | General Purpose, Analytics | Offline-first, Mobile Sync | Real-time apps, Dashboards |

## Deep Dive: Key Concepts for Seniors

### 1. CAP Theorem Placement
- **MongoDB (CP)**: Prioritizes consistency. If a primary fails, the system stops writes until a new one is elected.
- **CouchDB (AP)**: Prioritizes availability. Every node can accept writes. Network partitions don't stop the system; they just lead to eventual consistency.
- **RethinkDB (CP)**: Similar to MongoDB, it ensures that all reads/writes go through a primary for a given shard to maintain strict consistency.

### 2. Multi-Version Concurrency Control (MVCC)
Mostly associated with **CouchDB**. It avoids locking by versioning documents. This allows high concurrency where readers never block writers. In contrast, MongoDB uses a more traditional (but highly optimized) locking mechanism at the document level within WiredTiger.

### 3. Real-time Architecture
**RethinkDB** is the pioneer here with **Changefeeds**. Instead of the application polling `SELECT * FROM table WHERE updated_at > last_time`, RethinkDB pushes the change.
- **MongoDB** now offers similar functionality via **Change Streams** (since v3.6), which uses the OpLog.
- **CouchDB** has a `_changes` feed which is the basis for its replication protocol.

### 4. Sharding and Scaling
- **MongoDB**: Native sharding with "mongos" routers. Highly mature but complex to manage.
- **RethinkDB**: One of the most user-friendly sharding setups via a web UI.
- **CouchDB**: Scales via its replication protocol; clusters can be geographically distributed easily.

## Go Driver Integration Table

| Database | Official/Recommended Driver | Notable Feature |
| :--- | :--- | :--- |
| **MongoDB** | `go.mongodb.org/mongo-driver` | Type-safe BSON, Context support |
| **CouchDB** | `github.com/go-kivik/kivik` | Abstract interface, supports multiple drivers |
| **RethinkDB** | `github.com/rethinkdb/rethinkdb-go` | Native support for Changefeeds and ReQL |

## Summary of Senior Interview Focus
1. **Consistency vs Availability**: Explain why you'd choose CouchDB (AP) for a global mobile app vs MongoDB (CP) for a financial ledger.
2. **Durability**: Explain the role of the Journal (MongoDB) vs the append-only storage (CouchDB).
3. **Indexing Strategy**: When to use Compound vs Multikey indexes.
4. **Performance Tuning**: How to identify "slow queries" using the Profiler (MongoDB) and how to optimize them via indexing or aggregation reshaping.
