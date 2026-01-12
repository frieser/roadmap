---
---

## Summary
**Replication** (specifically Leader-Follower) is a technique used to scale **Read** capacity and provide **High Availability**. Data is written to a primary "Leader" node and copied to one or more "Follower" nodes. The followers serve read-only traffic.

## Detailed Explanation

### Workflow
1.  **Write**: The application sends `INSERT/UPDATE/DELETE` queries to the **Leader**.
2.  **Replicate**: The Leader sends the data changes (via WAL or Binary Log) to the **Followers**.
3.  **Read**: The application sends `SELECT` queries to any of the **Followers**.

### Sync vs. Async Replication
*   **Asynchronous**: The Leader confirms the write to the client immediately, without waiting for followers.
    *   *Pros*: Fast writes.
    *   *Cons*: **Replication Lag**. If the Leader crashes, recent data not yet copied to followers is lost.
*   **Synchronous**: The Leader waits for at least one follower to confirm the write.
    *   *Pros*: Zero data loss (Strong Consistency).
    *   *Cons*: Slower writes; if followers are down, the write fails.

### Scaling Reads
This pattern is ideal for **Read-Heavy** applications (like blogs, news sites, social media) where reads vastly outnumber writes. You can add more followers to handle more read traffic.

## Go Context: Connection Splitting
Using **GORM's DBResolver** to automatically route reads and writes.

```go
import (
	"gorm.io/gorm"
	"gorm.io/plugin/dbresolver"
	"gorm.io/driver/postgres"
)

func SetupDB() *gorm.DB {
	db, _ := gorm.Open(postgres.Open("host=leader_db ..."), &gorm.Config{})

	db.Use(dbresolver.Register(dbresolver.Config{
		// Traffic splitting
		Sources:  []gorm.Dialector{postgres.Open("host=leader_db ...")},
		Replicas: []gorm.Dialector{postgres.Open("host=follower_1 ..."), postgres.Open("host=follower_2 ...")},
		Policy:   dbresolver.RandomPolicy{},
	}))
	
	return db
}

// Usage
db.Create(&user) // Goes to Leader
db.Find(&users)  // Goes to a Follower
```

## Interview Questions

### Q: What is Replication Lag and how do you handle it?
**A:** Replication Lag is the delay between a write on the Leader and its appearance on the Follower. Users might update their profile and then immediately see the old data. To fix this, implement **Read-Your-Writes** consistency: ensure that for a short window after a write, the user reads from the Leader.

### Q: Can Replication scale Writes?
**A:** No. In a standard Leader-Follower setup, all writes must go to the single Leader. Adding followers does not increase write capacity (and might slightly decrease it due to replication overhead). For write scaling, you need Sharding or Multi-Leader replication.
