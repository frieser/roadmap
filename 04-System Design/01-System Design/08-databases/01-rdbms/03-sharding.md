---
---

## Summary
**Sharding** is a method of horizontal scaling where a single logical dataset is partitioned across multiple physical database servers (shards). Each shard holds a subset of the data, allowing the system to exceed the storage and throughput limits of a single machine.

## Detailed Explanation

### Sharding Strategies
1.  **Hash Sharding**: Uses a hash function on the Shard Key (e.g., `UserID`) to determine the shard.
    *   *Formula*: `ShardID = Hash(UserID) % TotalShards`
    *   *Pros*: Even distribution of data.
    *   *Cons*: Adding new shards requires re-hashing huge amounts of data (Resharding).
2.  **Range Sharding**: Divides data based on ranges of the key (e.g., Users A-M on Shard 1, N-Z on Shard 2).
    *   *Pros*: Efficient for range queries.
    *   *Cons*: **Hotspots**. If user IDs are sequential timestamps, all write traffic hits the newest shard.
3.  **Directory Based**: A lookup service maintains a map of `Key -> Shard`. Flexible but adds latency and a point of failure.

### Key Challenges
*   **Cross-Shard Joins**: extremely expensive or impossible. You usually have to denormalize data to avoid this.
*   **Resharding**: Moving data without downtime is complex.
*   **Unique IDs**: You can't use auto-incrementing IDs easily. You need global ID generators (e.g., Snowflake).

## Go Context: Routing Logic
A simple client-side sharding implementation.

```go
type ShardedClient struct {
	shards []*sql.DB
}

// GetShard determines which DB connection to use
func (c *ShardedClient) GetShard(key string) *sql.DB {
	hash := fnv.New32a()
	hash.Write([]byte(key))
	idx := hash.Sum32() % uint32(len(c.shards))
	return c.shards[idx]
}

func (c *ShardedClient) SaveUser(u User) {
	db := c.GetShard(u.ID)
	db.Exec("INSERT INTO users ...", ...)
}
```

## Interview Questions

### Q: What is a "Hot Key" problem?
**A:** Even with sharding, if a single key (e.g., "Justin Bieber" in a Twitter clone) receives a disproportionate amount of traffic, the specific shard hosting that key will be overwhelmed while others are idle. Solving this often requires caching or further splitting that specific key.

### Q: How do you perform a transaction that spans multiple shards?
**A:** Traditional ACID transactions don't work. You must use **Distributed Transactions** (like Two-Phase Commit), which are slow, or redesign the app to avoid them (e.g., using eventual consistency or grouping related data on the same shard via "Entity Groups").
