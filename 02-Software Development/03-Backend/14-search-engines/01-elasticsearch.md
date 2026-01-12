---
---

# Elasticsearch

## 1. Summary
Elasticsearch is a distributed, multitenant-capable full-text search engine with an HTTP web interface and schema-free JSON documents. It is based on the Apache Lucene library and is the core of the "ELK" stack (Elasticsearch, Logstash, Kibana).

## 2. Detailed Explanation
Elasticsearch is designed for horizontal scalability, reliability, and real-time search capabilities. It organizes data into **Indices**, which are further split into **Shards** distributed across a cluster of nodes.

### Key Concepts:
- **Document**: A basic unit of information (JSON).
- **Index**: A collection of documents with similar characteristics.
- **Inverted Index**: The underlying data structure that allows for fast full-text searches.
- **REST API**: All operations (indexing, searching, deleting) are performed via HTTP.

### Strengths:
- **Full-Text Search**: Complex queries like fuzzy matching, stemming, and phrase matching.
- **Aggregations**: Powerful analytics on top of search results.
- **Scalability**: Add nodes to the cluster to handle more data and traffic.

## 3. Go-specific Context & Examples
The official client for Go is `github.com/elastic/go-elasticsearch`. Modern versions (v8+) provide a **Typed API** which is much easier to use than the old low-level client.

### Example: Searching with Typed API
```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/elastic/go-elasticsearch/v8"
)

func main() {
	// 1. Initialize the client
	es, err := elasticsearch.NewTypedClient(elasticsearch.Config{
		Addresses: []string{"http://localhost:9200"},
	})
	if err != nil {
		log.Fatalf("Error creating client: %s", err)
	}

	// 2. Perform a Search
	res, err := es.Search().
		Index("products").
		Request(&search.Request{
			Query: &types.Query{
				Match: map[string]types.MatchQuery{
					"name": {Query: "Go Programming"},
				},
			},
		}).
		Do(context.Background())

	if err != nil {
		log.Fatalf("Error searching: %s", err)
	}

	fmt.Printf("Found %d hits\n", res.Hits.Total.Value)
}
```

## 4. Interview Questions
1. **How does an "Inverted Index" work?**
   - It maps terms (words) to the documents that contain them, allowing for fast lookup without scanning every document.
2. **What is a "Shard" in Elasticsearch?**
   - A shard is a single Lucene instance. Elasticsearch splits an index into multiple shards to distribute data across nodes.
3. **What is the difference between a "Term Query" and a "Match Query"?**
   - A Term Query looks for the exact word (not analyzed), whereas a Match Query analyzes the input text (e.g., lowercasing, stemming) before searching.
