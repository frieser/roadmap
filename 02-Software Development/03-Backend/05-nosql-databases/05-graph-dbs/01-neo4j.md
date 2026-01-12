---
---

## Summary
**Neo4j** is the world's leading **Graph Database**. It is a native graph store, meaning it is designed from the ground up to store and process data as a network of nodes and relationships. Unlike relational databases that use expensive JOIN operations, Neo4j uses "pointer chasing" to traverse relationships with constant-time performance, regardless of the size of the dataset.

## Detailed Explanation
Neo4j follows the **Property Graph Model**, which consists of:
*   **Nodes**: The entities in the graph (e.g., User, Product, City).
*   **Labels**: Used to group nodes (e.g., a node can have the label `:Person`).
*   **Relationships**: Connect nodes (e.g., `(User)-[:BOUGHT]->(Product)`). Relationships always have a direction and a type.
*   **Properties**: Key-value pairs stored on both nodes and relationships.

### Cypher Query Language
Neo4j uses **Cypher**, a declarative, pattern-matching query language that is highly intuitive.
Example: `MATCH (u:User {id: 1})-[:FRIEND]->(f) RETURN f.name`

### Key Features
*   **Native Graph Processing**: Relationships are stored as physical pointers, enabling "Index-free adjacency".
*   **ACID Compliant**: Unlike many NoSQL databases, Neo4j supports full ACID transactions.
*   **Scalability**: Supports causal clustering for high availability and read scaling.
*   **GDS (Graph Data Science)**: Library for running complex algorithms like PageRank, Louvain, and Pathfinding.

### Why use Neo4j?
1.  **Highly Connected Data**: When the relationships between data points are as important as the data itself (e.g., social networks, recommendation engines).
2.  **Pathfinding**: Finding the shortest path or identifying hidden patterns across multiple levels of connections.
3.  **Flexibility**: Schema-optional nature allows the graph to evolve as the business needs change.

## Go Application
Neo4j provides an official driver for Go that supports the Bolt protocol.

### Client Library
*   **Neo4j Go Driver**: `github.com/neo4j/neo4j-go-driver/v5`

### Go Example
```go
package main

import (
	"context"
	"fmt"

	"github.com/neo4j/neo4j-go-driver/v5/neo4j"
)

func main() {
	ctx := context.Background()
	dbUri := "bolt://localhost:7687"
	dbUser := "neo4j"
	dbPassword := "password"

	driver, err := neo4j.NewDriverWithContext(dbUri, neo4j.BasicAuth(dbUser, dbPassword, ""))
	if err != nil {
		panic(err)
	}
	defer driver.Close(ctx)

	// Run a query
	result, err := neo4j.ExecuteQuery(ctx, driver,
		"MATCH (p:Person) WHERE p.name = $name RETURN p.age AS age",
		map[string]any{"name": "Alice"},
		neo4j.EagerResultTransformer,
		neo4j.ExecuteQueryWithDatabaseParam("neo4j"),
	)
	if err != nil {
		panic(err)
	}

	for _, record := range result.Records {
		age, _ := record.Get("age")
		fmt.Printf("Alice is %v years old\n", age)
	}
}
```

## Interview Questions
**Q: What is "Index-free Adjacency" and why is it important for graph databases?**
**A:** Index-free adjacency means that every node in the graph maintains direct pointers to its adjacent nodes. This allows for relationship traversal without needing to perform a global index lookup for every step. This results in traversal performance that is independent of the total size of the graph.

**Q: Compare Neo4j with a Relational Database for a social network "Friend of a Friend" query.**
**A:** In a Relational DB, finding friends of friends requires multiple JOINs on a "Relationships" table, which becomes exponentially slower as the depth of the query increases. In Neo4j, this is a simple traversal across pointers, which is significantly faster and easier to write in Cypher.

**Q: Does Neo4j support ACID transactions?**
**A:** Yes, Neo4j is a fully ACID-compliant database. This makes it suitable for mission-critical applications where data integrity is paramount, distinguishing it from many other NoSQL databases that only offer eventual consistency.
