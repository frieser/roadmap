---
---

# Elastic Stack (ELK)

The Elastic Stack (formerly ELK Stack) is the leading open-source log management solution. It consists of **Elasticsearch** (search engine), **Logstash** (processing pipeline), **Kibana** (visualization), and **Beats** (lightweight shippers).

## Summary

ELK is designed for speed and flexibility. It indexes JSON documents in near real-time. DevOps engineers use it for everything from centralized logging to APM (Application Performance Monitoring). Its open architecture (REST API) makes it easy to integrate with any language.

## Detailed Explanation

### 1. Components
*   **Beats**: Lightweight agents (Filebeat, Metricbeat) installed on servers to ship data to Logstash or Elasticsearch.
*   **Logstash**: A heavy processing pipeline. It ingests data, filters/transforms it (Grok patterns), and outputs it to ES.
*   **Elasticsearch (ES)**: A distributed, RESTful search and analytics engine based on Lucene. It stores data in "Indices" composed of "Shards".
*   **Kibana**: The UI. Used to create dashboards, graphs, and search logs using KQL (Kibana Query Language).

### 2. Index Lifecycle Management (ILM)
Automates the management of indices over time (Hot -> Warm -> Cold -> Delete). Crucial for cost control in large clusters.

---

## Go Implementation Example

Using the official `github.com/elastic/go-elasticsearch/v8` client to index a log document.

```go
package main

import (
	"context"
	"log"
	"strings"

	"github.com/elastic/go-elasticsearch/v8"
	"github.com/elastic/go-elasticsearch/v8/esapi"
)

func main() {
	// 1. Initialize Client
	cfg := elasticsearch.Config{
		Addresses: []string{"http://localhost:9200"},
	}
	es, err := elasticsearch.NewClient(cfg)
	if err != nil {
		log.Fatalf("Error creating client: %s", err)
	}

	// 2. Define Log Document
	doc := `{"time": "2023-10-27T10:00:00Z", "level": "INFO", "msg": "User logged in", "service": "auth"}`

	// 3. Index Request
	req := esapi.IndexRequest{
		Index:      "app-logs-2023",
		DocumentID: "1", // Optional: Auto-generated if omitted
		Body:       strings.NewReader(doc),
		Refresh:    "true",
	}

	// 4. Perform Request
	res, err := req.Do(context.Background(), es)
	if err != nil {
		log.Fatalf("Error indexing: %s", err)
	}
	defer res.Body.Close()

	if res.IsError() {
		log.Printf("Indexing failed: %s", res.Status())
	} else {
		log.Println("Log indexed successfully into Elasticsearch")
	}
}
```

## Interview Questions

**Q: What is a Shard in Elasticsearch and why is it important?**
**A:** A Shard is a self-contained Lucene index. An Elasticsearch Index is actually a logical grouping of one or more physical shards distributed across the cluster nodes. Sharding allows ES to split data volume and parallelize operations. However, too many small shards ("oversharding") consume excessive heap memory and slow down the cluster.

**Q: Explain the role of Logstash vs Beats.**
**A:** **Beats** are lightweight shippers written in Go, designed to run on edge servers with minimal resource usage (just shipping data). **Logstash** is a heavy Java application designed for complex data transformation (parsing, enrichment, filtering). A common pattern is `Beats -> Logstash (Processing) -> Elasticsearch`.

**Q: What is the "Split Brain" problem in Elasticsearch?**
**A:** It occurs when a network partition causes nodes to elect two different Masters, leading to data divergence (two conflicting versions of the truth). This is prevented by configuring `discovery.zen.minimum_master_nodes` (in older versions) or relying on the modern Raft-based consensus algorithm (in 7.x+) which requires a quorum (N/2 + 1) to elect a master.
