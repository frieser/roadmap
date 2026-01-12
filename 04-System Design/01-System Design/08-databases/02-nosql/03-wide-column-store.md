---
---

## Summary
Wide-Column stores (or Column-Family databases) store data in tables, rows, and dynamic columns. Unlike traditional RDBMS, columns are stored together on disk rather than rows. This makes them exceptionally efficient for high-write throughput and large-scale data distribution across multiple data centers.

## Detailed Explanation

### Core Concepts
*   **Column Families**: Related columns are grouped together.
*   **Sparse Data**: Does not store "null" values for missing columns, making it space-efficient for sparse datasets.
*   **High Write Throughput**: Optimized for appending data rather than complex updates.
*   **Partitioning**: Data is partitioned based on a partition key, allowing it to be spread across a cluster.

### Use Cases
1.  **Time-Series Data**: Storing metrics, logs, or stock price data where writes are constant.
2.  **IoT Telemetry**: Handling massive streams of data from millions of devices.
3.  **Large-scale Web Analytics**: Tracking user behavior across massive platforms (e.g., Facebook, Netflix).
4.  **Messaging Systems**: Storing message history (e.g., Discord uses Cassandra).

### Notable Examples
*   **Apache Cassandra**: A distributed, decentralized (masterless) wide-column store.
*   **HBase**: Built on top of HDFS (Hadoop), modeled after Google's BigTable.
*   **ScyllaDB**: A C++ rewrite of Cassandra, designed for ultra-low latency.

### Go Application
Using Cassandra with the `gocql` library.

```go
package main

import (
	"fmt"
	"github.com/gocql/gocql"
	"log"
)

func main() {
	cluster := gocql.NewCluster("127.0.0.1")
	cluster.Keyspace = "iot_data"
	session, _ := cluster.CreateSession()
	defer session.Close()

	// Insert time-series data
	if err := session.Query(`INSERT INTO sensor_readings (sensor_id, ts, value) VALUES (?, ?, ?)`,
		"sensor_01", gocql.TimeUUID(), 23.5).Exec(); err != nil {
		log.Fatal(err)
	}

	// Query data for a specific sensor
	var sensorID string
	var value float64
	iter := session.Query(`SELECT sensor_id, value FROM sensor_readings WHERE sensor_id = ?`, "sensor_01").Iter()
	for iter.Scan(&sensorID, &value) {
		fmt.Printf("Sensor: %s, Temp: %f\n", sensorID, value)
	}
}
```

## Interview Questions

**Q: How does Cassandra achieve its high write throughput?**
**A:** Cassandra writes data to an in-memory structure called a **MemTable** and appends it to a **CommitLog** (on disk) for durability. Once the MemTable is full, it is flushed to disk as an **SSTable**. This process is purely sequential, avoiding expensive disk seeks.

**Q: What is a "Compaction" in wide-column stores?**
**A:** Compaction is the background process of merging multiple SSTables into a single one, while removing deleted data (using tombstones) and reconciling different versions of the same record.

**Q: Explain the difference between Row-oriented (SQL) and Column-oriented storage.**
**A:** Row-oriented storage (PostgreSQL, MySQL) stores all columns of a row together. Good for OLTP (Online Transactional Processing). Column-oriented storage stores all values of a specific column together. This is great for analytical queries (OLAP) where you only need a few columns but millions of rows.
