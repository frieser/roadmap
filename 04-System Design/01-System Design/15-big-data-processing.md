---
topic: Big Data Processing (Hadoop, Spark, MapReduce)
category: System Design
tags: [big-data, hadoop, spark, mapreduce, distributed-systems]
---

## Summary

The Big Data ecosystem has evolved from the rigid, disk-heavy paradigms of **Apache Hadoop (MapReduce)** to the flexible, memory-optimized world of **Apache Spark**. While Hadoop provided the foundation for distributed storage (HDFS) and resource management (YARN), Spark revolutionized data processing by utilizing in-memory computation and Directed Acyclic Graphs (DAGs). Modern architects must choose between these self-managed frameworks and cloud-native serverless solutions like **BigQuery** or **Snowflake** based on operational complexity, cost, and data latency requirements.

## Detailed Explanation

### 1. Evolution: From Disk to Memory

The transition from MapReduce to Spark represents the most significant shift in distributed computing history.

*   **MapReduce (The Disk Era):**
    *   **Architecture:** Jobs are divided into `Map` and `Reduce` phases.
    *   **Bottleneck:** Every step (Map, Shuffle, Reduce) persists intermediate data to **HDFS**. This results in high I/O latency.
    *   **Use Case:** Batch processing where time is not critical (e.g., overnight log processing).
*   **Apache Spark (The Memory Era):**
    *   **Architecture:** Uses **Resilient Distributed Datasets (RDDs)** and **DataFrames**.
    *   **Optimization:** Keeps intermediate data in **RAM**. It only writes to disk when memory is full or for final persistence.
    *   **Performance:** Can be up to 100x faster than MapReduce for iterative algorithms (like Machine Learning).

### 2. Core Architecture Components

#### Hadoop Ecosystem
*   **HDFS (Hadoop Distributed File System):** A distributed, fault-tolerant storage system that splits files into large blocks (typically 128MB) and replicates them across a cluster.
*   **YARN (Yet Another Resource Negotiator):** The "Operating System" of Hadoop. It manages cluster resources and schedules jobs.
*   **MapReduce:** The software framework for writing applications that process vast amounts of data in parallel.

#### Spark Ecosystem
*   **Spark Core:** The underlying execution engine that provides distributed task scheduling and RDD abstractions.
*   **Spark SQL:** High-level API for structured data, allowing users to run SQL queries alongside programmatic data transformations.
*   **Spark Streaming:** Processes real-time data using "micro-batches," enabling low-latency stream processing.

### 3. Batch vs. Stream Processing

| Feature | Batch Processing (Hadoop/Spark) | Stream Processing (Spark Streaming/Flink) |
| :--- | :--- | :--- |
| **Data Scope** | Processes all data in a large dataset. | Processes data as it arrives (infinite). |
| **Latency** | High (Minutes to Hours). | Low (Milliseconds to Seconds). |
| **Use Case** | Financial audit, ETL, Historical analysis. | Fraud detection, Real-time dashboards. |

### 4. Selection Criteria: Big Data vs. Modern Alternatives

As a Software Architect, deciding between a self-managed Hadoop/Spark cluster and a cloud-native tool depends on:

*   **When to choose Hadoop/Spark:**
    *   **Data Sovereignty:** If data cannot leave on-premise servers.
    *   **Complex Custom Logic:** If transformations require complex Java/Scala/Python code that SQL cannot handle efficiently.
    *   **Cost Control:** If you have existing hardware and a dedicated team to manage YARN/HDFS.
*   **When to choose Cloud-Native (BigQuery/Snowflake):**
    *   **Zero Management:** If you want a "Serverless" experience with no clusters to scale.
    *   **Ad-hoc Analysis:** If analysts need to run massive SQL queries occasionally without maintaining idle infrastructure.
    *   **Pay-per-query:** Better for sporadic workloads compared to 24/7 cluster costs.

## Go Code Examples

In Go, we can simulate the MapReduce paradigm using goroutines and channels to process data concurrently. This demonstrates the core principle of "Divide and Conquer."

### Basic MapReduce Implementation

```go
package main

import (
	"fmt"
	"strings"
	"sync"
)

// Map function: emits key-value pairs
func mapper(text string, ch chan<- map[string]int) {
	counts := make(map[string]int)
	words := strings.Fields(text)
	for _, word := range words {
		counts[strings.ToLower(word)]++
	}
	ch <- counts
}

// Reduce function: aggregates counts
func reducer(results []map[string]int) map[string]int {
	finalCounts := make(map[string]int)
	for _, res := range results {
		for word, count := range res {
			finalCounts[word] += count
		}
	}
	return finalCounts
}

func main() {
	inputs := []string{
		"Hadoop is disk based",
		"Spark is memory based",
		"Spark is faster than Hadoop",
	}

	ch := make(chan map[string]int, len(inputs))
	var wg sync.WaitGroup

	// MAP PHASE
	for _, input := range inputs {
		wg.Add(1)
		go func(i string) {
			defer wg.Done()
			mapper(i, ch)
		}(input)
	}

	wg.Wait()
	close(ch)

	// Collect intermediate results
	var intermediate []map[string]int
	for res := range ch {
		intermediate = append(intermediate, res)
	}

	// REDUCE PHASE
	finalResult := reducer(intermediate)

	fmt.Println("Word Counts:")
	for word, count := range finalResult {
		fmt.Printf("%s: %d\n", word, count)
	}
}
```

### Go Application: Interacting with HDFS

For a real-world Go application, you would use a library like `colinmarc/hdfs` to interact with a Hadoop cluster.

```go
// Example (Conceptual)
// client, _ := hdfs.New("localhost:9000")
// file, _ := client.Open("/data/logs.txt")
// io.Copy(os.Stdout, file)
```

## Interview Questions

**Q: Why is Spark generally faster than MapReduce?**
**A:** Spark minimizes disk I/O by keeping intermediate data in memory (RAM). It also uses a DAG (Directed Acyclic Graph) to optimize the execution plan and combine multiple stages, whereas MapReduce must write to HDFS between every Map and Reduce stage.

**Q: Explain the role of YARN in a Hadoop cluster.**
**A:** YARN (Yet Another Resource Negotiator) acts as the resource management layer. It separates the resource management (managing CPU/Memory across the cluster) from the data processing logic, allowing different engines (MapReduce, Spark, Tez) to run on the same shared hardware.

**Q: When would you use a standard relational database instead of a Big Data framework?**
**A:** Use a relational database (like Postgres) when you need low-latency point reads/writes (OLTP), strong ACID compliance for small transactions, and the data volume fits within a few terabytes. Big Data frameworks are designed for high-throughput analytical scans (OLAP) on petabytes of unstructured data.

**Q: What is the "Data Locality" principle in Hadoop?**
**A:** Data Locality means moving the computation code to the node where the data is stored, rather than moving large amounts of data over the network to the computation node. This significantly reduces network congestion in large clusters.
