---
---

## Summary
**Apache HBase** is an open-source, distributed, versioned, non-relational database modeled after Google's Bigtable. It is built on top of the **Hadoop Distributed File System (HDFS)** and is designed to provide random, real-time read/write access to Big Data. Unlike Cassandra, HBase is **strongly consistent** and follows the CP (Consistency and Partition Tolerance) model of the CAP theorem.

## Detailed Explanation
HBase is part of the Apache Hadoop ecosystem and is ideal for hosting very large tables (billions of rows, millions of columns).

### Architecture
*   **HMaster**: Coordinates the cluster, handles schema changes, and manages RegionServer assignments.
*   **RegionServer**: Handles read and write requests for a set of "Regions". A Region is a subset of a table's data.
*   **Zookeeper**: Used for coordination, master election, and tracking RegionServer health.
*   **HDFS**: The underlying storage layer where data is stored as HFiles.

### Data Model
*   **Row Key**: The primary index, sorted lexicographically. Good row key design is critical for performance to avoid "hotspotting".
*   **Column Family**: Grouping of columns. All members of a column family are stored together in the same HFile.
*   **Qualifier**: The specific column within a family (e.g., `info:name`, `info:email`).
*   **Timestamp**: Allows for versioning. By default, HBase keeps multiple versions of a cell value.

### Consistency vs Availability
HBase prioritizes **Consistency**. If a RegionServer goes down, the data it served becomes unavailable until the HMaster reassigns that region and it is recovered. This is a key difference from Cassandra's "Available" (AP) model.

## Go Application
For Go, the most mature client is `gohbase`, which is a pure-Go implementation.

### Client Library
*   **gohbase**: `github.com/tsuna/gohbase`

### Go Example
```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/tsuna/gohbase"
	"github.com/tsuna/gohbase/hrpc"
)

func main() {
	// Create a client
	client := gohbase.NewClient("localhost") // Connects to Zookeeper

	// Put some data
	values := map[string]map[string][]byte{
		"info": {
			"name": []byte("Alice"),
		},
	}
	putRequest, err := hrpc.NewPutStr(context.Background(), "my_table", "row_key_1", values)
	if err != nil {
		log.Fatal(err)
	}
	_, err = client.Put(putRequest)
	if err != nil {
		log.Fatal(err)
	}

	// Get data
	getRequest, err := hrpc.NewGetStr(context.Background(), "my_table", "row_key_1")
	if err != nil {
		log.Fatal(err)
	}
	getRsp, err := client.Get(getRequest)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("Result: %v\n", getRsp.Cells)
}
```

## Interview Questions
**Q: How does HBase differ from Cassandra in terms of the CAP theorem?**
**A:** HBase is a **CP** system (Consistency and Partition Tolerance). It ensures that a read always returns the most recent write, but if a master or region server fails, parts of the data might be temporarily unavailable. Cassandra is an **AP** system (Availability and Partition Tolerance), ensuring high availability at the cost of eventual consistency.

**Q: Why is "Row Key" design so important in HBase?**
**A:** Because HBase stores rows sorted by the Row Key and partitions them into regions based on key ranges. If many requests target keys that are close together (e.g., using a timestamp as the prefix), a single RegionServer will be overwhelmed while others are idle. This is called **hotspotting**. Good designs often involve salting or reversing keys to distribute load.

**Q: What is the role of Zookeeper in an HBase cluster?**
**A:** Zookeeper acts as the coordination service. it is used for:
1.  **Master Election**: Ensuring only one HMaster is active.
2.  **Service Discovery**: Helping clients find which RegionServer is hosting a particular region.
3.  **Health Monitoring**: Detecting when a RegionServer fails so the Master can reassign its regions.
