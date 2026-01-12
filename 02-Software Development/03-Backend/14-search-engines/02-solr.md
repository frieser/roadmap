---
---

# Apache Solr

## 1. Summary
Apache Solr is an open-source enterprise search platform built on Apache Lucene. It is highly reliable, scalable, and fault-tolerant, providing distributed indexing, replication, and load-balanced querying.

## 2. Detailed Explanation
While Elasticsearch is often preferred for logging and dynamic data, Solr is traditionally favored for enterprise search applications where stability and complex configuration (via XML) are preferred.

### Key Features:
- **Advanced Text Analysis**: Supports complex linguistic analysis.
- **Faceted Search**: Allows users to filter results by categories (e.g., price range, brand).
- **Schema Design**: Historically used `schema.xml` to define fields strictly, though it now supports schemaless modes.
- **SolrCloud**: The distributed mode of Solr that provides high availability and scaling.

## 3. Go-specific Context & Examples
There is no "official" Solr client for Go from the Apache project. Developers typically use third-party libraries like `github.com/vanng822/go-solr` or interact with the JSON API directly using `net/http`.

### Example: Querying Solr via HTTP
```go
package main

import (
    "io"
    "net/http"
    "fmt"
)

func main() {
    url := "http://localhost:8983/solr/my_collection/select?q=name:Go&wt=json"
    
    resp, err := http.Get(url)
    if err != nil {
        panic(err)
    }
    defer resp.Body.Close()
    
    body, _ := io.ReadAll(resp.Body)
    fmt.Println(string(body))
}
```

## 4. Interview Questions
1. **What is the main difference between Solr and Elasticsearch?**
   - Solr is often more mature for static enterprise search with complex schemas; Elasticsearch is more user-friendly for dynamic data and logging (ELK).
2. **How does Solr handle high availability?**
   - Via **SolrCloud**, which uses ZooKeeper for coordination and provides replication of shards.
3. **What is "Faceting" in search engines?**
   - It is the process of categorizing search results into groups (facets) to help users narrow down their search.
