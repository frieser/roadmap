---
---

## Summary
Sharding is a method of horizontal scaling that involves splitting a large database into smaller, faster, and more easily managed parts called "shards." Each shard is a separate database instance containing a subset of the total data.

## Detailed Explanation
Unlike replication (where every node has the same data), in sharding, each node has unique data.

### Sharding Strategies
1.  **Key-based (Hash) Sharding**: A hash function is applied to a "sharding key" (like UserID) to determine which shard the data belongs to. Provides even distribution.
2.  **Range-based Sharding**: Data is split based on ranges of a value (e.g., Users A-M on Shard 1, N-Z on Shard 2). Simple but can lead to "hot spots."
3.  **Directory-based Sharding**: A lookup service keeps track of which data is on which shard. Flexible but adds a single point of failure.

### Challenges of Sharding
-   **Complexity**: The application must know how to route queries to the correct shard.
-   **No Cross-shard Joins**: Joining tables that reside on different shards is extremely difficult and slow.
-   **Rebalancing**: Moving data between shards as they grow is a complex operation.

## Go-specific Context
Implementing sharding in Go can be done at the application level or using middleware like Vitess (for MySQL).

### Application-level Sharding in GORM
```go
import (
    "gorm.io/sharding"
)

// Register sharding for a specific table
db.Use(sharding.Register(sharding.Config{
    ShardingKey:         "user_id",
    NumberOfShards:      64,
    PrimaryKeyGenerator: sharding.PKSnowflake,
}, "orders", "notifications"))

// GORM will now automatically route queries to the correct table/db shard
db.Create(&Order{UserID: 100}) 
```

## Interview Questions
**Q: When should you consider Sharding over Replication?**
**A:** Replication scales **reads**. Sharding scales **writes** and total storage capacity. You consider sharding when your primary database can no longer handle the volume of incoming writes or the dataset is too large for a single machine.

**Q: What is a 'Hot Spot' in sharding?**
**A:** A hot spot occurs when one shard receives significantly more traffic than others. For example, if you shard by "Date" and all new users are created today, the "Today" shard will be overwhelmed while older shards remain idle.

**Q: What is 'Horizontal Scaling' vs 'Vertical Scaling'?**
**A:** Vertical scaling (Scaling Up) means adding more power (CPU, RAM) to an existing server. Horizontal scaling (Scaling Out) means adding more servers to the system. Sharding is a horizontal scaling technique.
