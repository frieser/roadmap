---
---

## Summary
RethinkDB is an open-source, scalable JSON database built for the real-time web. Unlike traditional databases that require polling for changes, RethinkDB can push updated query results to applications in real-time using **Changefeeds**. It features a functional, chainable query language called **ReQL**.

## Detailed Explanation

### Real-time Push Architecture (Changefeeds)
RethinkDB "inverts" the traditional database model. Instead of the application polling the database for changes, the application can subscribe to a feed.
- **Changefeeds**: You can call `.changes()` on a table, a selection, or even an aggregation. RethinkDB will then keep a persistent connection open and push any changes (newly inserted, updated, or deleted documents) to the client as they happen.
- This is significantly more efficient than polling and simplifies the development of real-time features like live dashboards, chats, and collaborative tools.

### ReQL (RethinkDB Query Language)
ReQL is a powerful, data-driven query language that is embedded in the programming language (DSL).
- It is **functional and chainable**: You build queries by chaining methods together (e.g., `r.table('users').filter({group: 'admin'}).map(func...).run()`).
- ReQL queries are executed entirely on the server, but they feel like native code in Go, Python, or JavaScript.
- It supports complex operations like distributed joins, subqueries, and geospatial queries natively.

### Architecture and Scaling
- **Clustering**: RethinkDB was designed for easy clustering. You can set up a cluster with a few clicks in the web UI.
- **Sharding and Replication**: It supports automatic sharding and replication. You can configure the number of shards and replicas per table.
- **Consistency**: RethinkDB is a **CP** system (Consistent and Partition Tolerant) by default. It uses a primary-secondary replication model where each shard has a primary that coordinates writes to ensure consistency.

### Go Integration
The most popular driver for Go is [rethinkdb-go](https://github.com/rethinkdb/rethinkdb-go).

```go
package main

import (
	"fmt"
	"log"

	r "github.com/rethinkdb/rethinkdb-go"
)

func main() {
	session, err := r.Connect(r.ConnectOpts{
		Address: "localhost:28015",
	})
	if err != nil {
		log.Fatalln(err)
	}

	// Insert data
	_, err = r.Table("users").Insert(map[string]string{
		"name":  "Alice",
		"email": "alice@example.com",
	}).RunWrite(session)

	// Subscribe to Changefeeds
	res, err := r.Table("users").Changes().Run(session)
	if err != nil {
		log.Fatalln(err)
	}

	var change map[string]interface{}
	fmt.Println("Waiting for changes...")
	for res.Next(&change) {
		fmt.Printf("Change detected: %v\n", change)
	}
}
```

## Interview Questions

**Q: What are Changefeeds in RethinkDB and why are they useful?**
**A:** Changefeeds allow an application to receive real-time updates whenever data in a table or query result changes. This eliminates the need for expensive polling mechanisms and makes it much easier to build reactive, real-time applications with minimal latency.

**Q: How does ReQL differ from SQL?**
**A:** SQL is a declarative string-based language. ReQL is a functional, chainable API (DSL) that is integrated into the host programming language. This makes ReQL queries easier to build dynamically and provides better type safety and IDE support compared to raw SQL strings.

**Q: Where does RethinkDB sit in the CAP Theorem?**
**A:** RethinkDB is primarily a **CP** system. It prioritizes consistency. Each shard has a designated primary node. If a partition occurs and the primary is on the minority side, writes to that shard will fail until a new primary is elected on the majority side, ensuring data consistency across the cluster.

**Q: Does RethinkDB support Joins?**
**A:** Yes, RethinkDB supports distributed joins natively. Unlike many NoSQL databases that discourage joins, ReQL provides a `.eqJoin()` command that allows you to efficiently join tables based on keys, even if the data is distributed across different shards.
