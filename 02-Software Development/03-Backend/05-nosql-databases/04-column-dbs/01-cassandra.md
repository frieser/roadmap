---
---

## Summary
**Apache Cassandra** is a highly scalable, high-performance distributed NoSQL database designed to handle large amounts of data across many commodity servers. It is a **wide-column store** that provides high availability with no single point of failure. It is based on a masterless architecture where all nodes are equal.

## Detailed Explanation
Cassandra combines the distributed design of Amazon's Dynamo with the data model of Google's Bigtable.

### Architecture
*   **Masterless (Peer-to-Peer)**: Any node can handle any request. There is no "leader" node, which eliminates single points of failure.
*   **Gossip Protocol**: Nodes communicate with each other to share state and health information.
*   **Partitioner**: Determines how data is distributed across the cluster (the "ring") based on a hash of the Partition Key.
*   **Replication Factor (RF)**: The number of nodes where a row of data is stored.
*   **Consistency Level**: Configurable per query (e.g., ONE, QUORUM, ALL), allowing developers to tune the balance between availability and consistency.

### Data Model
*   **Keyspace**: Similar to a schema/database in SQL.
*   **Table**: A collection of rows.
*   **Partition Key**: Determines which node stores the data.
*   **Clustering Key**: Determines the sorting order of data within a partition.
*   Combined, they form the **Primary Key**.

### Write Path (LSM Tree based)
1.  **Commit Log**: Data is first written here for durability.
2.  **Memtable**: Data is then written to an in-memory buffer.
3.  **SSTable**: When the memtable is full, it is flushed to disk as an immutable Sorted String Table (SSTable).
4.  **Compaction**: Background process that merges SSTables and removes deleted data (tombstones).

## Go Application
The community standard for connecting to Cassandra from Go is `gocql`.

### Client Library
*   **gocql**: `github.com/gocql/gocql`

### Go Example
```go
package main

import (
	"fmt"
	"log"

	"github.com/gocql/gocql"
)

func main() {
	// Connect to the cluster
	cluster := gocql.NewCluster("127.0.0.1")
	cluster.Keyspace = "example"
	cluster.Consistency = gocql.Quorum
	session, err := cluster.CreateSession()
	if err != nil {
		log.Fatal(err)
	}
	defer session.Close()

	// Insert data
	if err := session.Query(`INSERT INTO users (id, name, email) VALUES (?, ?, ?)`,
		gocql.TimeUUID(), "John Doe", "john@example.com").Exec(); err != nil {
		log.Fatal(err)
	}

	// Query data
	var id gocql.UUID
	var name, email string
	if err := session.Query(`SELECT id, name, email FROM users WHERE name = ? ALLOW FILTERING`,
		"John Doe").Scan(&id, &name, &email); err != nil {
		log.Fatal(err)
	}
	fmt.Printf("User: %s <%s>\n", name, email)
}
```

## Interview Questions
**Q: What is the "Eventual Consistency" in Cassandra and how can you achieve "Strong Consistency"?**
**A:** By default, Cassandra is eventually consistent, meaning data will eventually propagate to all replicas. To achieve strong consistency (Read-Your-Writes), you must ensure that `R + W > RF` (Read Level + Write Level > Replication Factor). For example, with an RF of 3, using `QUORUM` (2) for both reads and writes ensures strong consistency.

**Q: What is the difference between a Partition Key and a Clustering Key?**
**A:** The Partition Key determines which node in the cluster will store the data (data distribution). The Clustering Key determines the physical sorting of data within that partition on the disk (data ordering). Together, they allow for efficient retrieval of sorted data within a specific partition.

**Q: Why is "ALLOW FILTERING" generally discouraged in Cassandra production queries?**
**A:** `ALLOW FILTERING` tells Cassandra that the query requires scanning all partitions in the table because the filter is not on a partition key. This can be extremely slow and resource-intensive in a distributed environment as it might involve every node in the cluster. It is better to model your data specifically for the queries you need to run.
