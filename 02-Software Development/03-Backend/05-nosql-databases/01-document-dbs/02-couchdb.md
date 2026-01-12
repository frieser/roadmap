---
---

## Summary
Apache CouchDB is a document-oriented NoSQL database that prioritizes availability and partition tolerance (AP in CAP). It is famous for its robust sync protocol, allowing seamless master-master replication between clusters and even mobile devices. CouchDB uses JSON for data, HTTP for the API, and JavaScript for MapReduce views.

## Detailed Explanation

### Multi-Version Concurrency Control (MVCC)
CouchDB uses **MVCC** to manage concurrent access to documents.
- Instead of locking a document during an update, CouchDB creates a new revision of the document.
- Each document has a `_rev` (revision ID). When updating, the client must provide the current `_rev`.
- If another update happened in the meantime, the `_rev` won't match, and the update will be rejected (Conflict).
- This ensures that readers never wait for writers and vice versa, providing high performance for read-heavy workloads.

### Replication Protocol (Master-Master)
CouchDB's core strength is its replication.
- It supports **Master-Master replication**, meaning any node can accept writes, and changes will be propagated to other nodes eventually.
- **Eventual Consistency**: Conflicts are not resolved automatically during replication but are flagged. CouchDB uses a deterministic algorithm to pick a "winning" revision, but all conflicting revisions are stored so the application can resolve them later.
- This makes CouchDB ideal for "Offline First" applications where devices might be disconnected for long periods.

### MapReduce Views
CouchDB doesn't use dynamic ad-hoc queries like SQL. Instead, it uses **Views** defined by MapReduce functions (typically written in JavaScript).
- **Map**: Filters and transforms documents into a set of key-value pairs.
- **Reduce**: Summarizes the mapped data (e.g., sum, count).
- Views are **pre-computed** and updated incrementally as documents change. This makes querying extremely fast but requires defining indexes upfront.

### Go Integration
Since CouchDB uses a RESTful HTTP API, any HTTP client can interact with it. There are also libraries like `kivik` that provide a more structured Go interface.

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/go-kivik/kivik/v3"
	_ "github.com/go-kivik/couchdb/v3" // CouchDB driver
)

func main() {
	client, err := kivik.New("couch", "http://admin:password@localhost:5984/")
	if err != nil {
		log.Fatal(err)
	}

	db := client.DB(context.TODO(), "testdb")

	// Create a document
	doc := map[string]interface{}{
		"_id":   "user_123",
		"name":  "Jane Doe",
		"email": "jane@example.com",
	}
	rev, err := db.Put(context.TODO(), "user_123", doc)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("Created revision:", rev)

	// Fetch document
	var fetchedDoc map[string]interface{}
	row := db.Get(context.TODO(), "user_123")
	if err := row.ScanDoc(&fetchedDoc); err != nil {
		log.Fatal(err)
	}
	fmt.Println("Fetched User:", fetchedDoc["name"])
}
```

## Interview Questions

**Q: How does CouchDB handle write conflicts during replication?**
**A:** CouchDB does not fail a replication when a conflict occurs. Instead, it stores both versions of the document and marks it as being in a "conflict" state. It deterministically chooses one as the "winner" so all nodes show the same data, but the application can query for conflicts and resolve them (e.g., merging data) manually.

**Q: Where does CouchDB sit in the CAP Theorem?**
**A:** CouchDB is an **AP** system (Available and Partition Tolerant). It ensures the system is always available for reads and writes even during network partitions. It achieves this through eventual consistency and its robust replication protocol.

**Q: What is the benefit of MVCC over traditional Locking?**
**A:** In a locking system, a writer blocks readers, and multiple writers block each other. In MVCC, readers always see the last consistent revision without being blocked by writers. This significantly increases concurrency and performance in distributed environments.

**Q: Why are CouchDB views pre-computed?**
**A:** Pre-computing views (MapReduce) ensures that query performance remains constant regardless of the amount of data, as the "heavy lifting" is done when data is written or updated, not when it's queried. This follows the philosophy of "efficient reads at the cost of slightly slower writes/storage."
