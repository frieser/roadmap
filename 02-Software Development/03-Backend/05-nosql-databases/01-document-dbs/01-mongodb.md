---
---

## Summary
MongoDB is a leading NoSQL, document-oriented database designed for high availability, horizontal scalability, and developer productivity. It stores data in flexible, JSON-like BSON documents, making it ideal for applications with evolving schemas. Key features include a powerful aggregation framework, native sharding, and ACID transaction support since version 4.0.

## Detailed Explanation

### Architecture: WiredTiger & Journaling
MongoDB uses **WiredTiger** as its default storage engine. It provides document-level concurrency control, checkpointing, and compression.
- **Checkpoints**: WiredTiger writes data to disk at regular intervals (every 60 seconds or 2GB of data), providing a consistent view.
- **Journaling**: To ensure durability between checkpoints, MongoDB uses a write-ahead log (WAL) called the **Journal**. In the event of a crash, MongoDB replays the journal to recover any writes that occurred after the last checkpoint.

### Replication vs. Sharding
- **Replication (Replica Sets)**: Focuses on **High Availability**. A replica set consists of one Primary node and multiple Secondary nodes. Data is asynchronously replicated from Primary to Secondaries. If the Primary fails, an election promotes a Secondary.
- **Sharding**: Focuses on **Horizontal Scalability**. It distributes data across multiple machines (shards). A shard key is used to determine which shard a document belongs to. Sharding is managed by `mongos` query routers and config servers.

### Aggregation Pipeline
The Aggregation Pipeline is a framework for data transformation. It processes documents through a multi-stage pipeline:
- `$match`: Filters documents (like SQL `WHERE`).
- `$group`: Groups documents by a specified key and performs aggregations (like SQL `GROUP BY`).
- `$project`: Reshapes documents (renaming fields, calculating new ones).
- `$lookup`: Performs left outer joins with other collections.
- `$unwind`: Deconstructs an array field from the input documents.

### ACID Transactions
Since version 4.0, MongoDB supports multi-document **ACID transactions**. This allows for atomic operations across multiple documents, collections, and even shards (v4.2+). Transactions use snapshot isolation, ensuring a consistent view of data.

### Indexing Types
MongoDB offers several indexing strategies:
- **Single Field**: Index on a single field.
- **Compound**: Index on multiple fields (order matters for prefix matching).
- **Multikey**: Indexing array fields.
- **Text**: For full-text search.
- **Geospatial**: For location-based queries (`2dsphere`, `2d`).
- **Hashed**: For hash-based sharding.

### Go Integration
The official [mongo-go-driver](https://github.com/mongodb/mongo-go-driver) is the standard for Go applications.

```go
package main

import (
	"context"
	"fmt"
	"log"
	"time"

	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)

type User struct {
	Name  string `bson:"name"`
	Email string `bson:"email"`
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()

	client, err := mongo.Connect(ctx, options.Client().ApplyURI("mongodb://localhost:27017"))
	if err != nil {
		log.Fatal(err)
	}

	collection := client.Database("testdb").Collection("users")

	// Insert
	res, err := collection.InsertOne(ctx, User{Name: "John Doe", Email: "john@example.com"})
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("Inserted ID:", res.InsertedID)

	// Query with Aggregation
	pipeline := mongo.Pipeline{
		{{"$match", bson.D{{"name", "John Doe"}}}},
		{{"$project", bson.D{{"name", 1}, {"email", 1}}}},
	}
	cursor, err := collection.Aggregate(ctx, pipeline)
	if err != nil {
		log.Fatal(err)
	}
	var results []bson.M
	if err = cursor.All(ctx, &results); err != nil {
		log.Fatal(err)
	}
	fmt.Println("Aggregated Results:", results)
}
```

## Interview Questions

**Q: MongoDB vs. SQL: When would you choose one over the other?**
**A:** Choose MongoDB for unstructured/evolving data, rapid prototyping, and horizontal scaling needs. Choose SQL for highly structured data, complex relational queries (joins), and when strict schema enforcement is a priority from the start.

**Q: Where does MongoDB sit in the CAP Theorem?**
**A:** By default, MongoDB is a **CP** system (Consistent and Partition Tolerant). It prioritizes consistency by having a single Primary. During a network partition, if the Primary is disconnected, the system becomes unavailable for writes until a new Primary is elected. However, consistency can be tuned (Read/Write Concerns) to lean towards AP.

**Q: What is the purpose of the Journal in WiredTiger?**
**A:** The Journal provides durability. Since data is only flushed to data files during checkpoints (every 60s), a crash could lose up to 60s of data. The Journal records every write operation immediately to disk, allowing MongoDB to recover the state by replaying the log after a crash.

**Q: Explain the "Covered Query" in MongoDB.**
**A:** A covered query is a query where all requested fields are part of an index, and the criteria also use the same index. In this case, MongoDB returns results directly from the index without ever looking at the actual documents (fetching from disk/RAM), which is extremely fast.
