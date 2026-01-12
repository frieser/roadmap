---
---

## Summary
GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data. It provides a complete and understandable description of the data in your API, giving clients the power to ask for exactly what they need and nothing more.

## Detailed Explanation
Developed by Facebook in 2012 and released publicly in 2015, GraphQL was designed to solve the problems of over-fetching and under-fetching data in REST APIs.

### Core Concepts
- **Schema**: A strong type system that defines the data available and its relationships.
- **Queries**: Requests made by the client to retrieve data.
- **Mutations**: Requests made to create, update, or delete data.
- **Subscriptions**: Real-time updates via WebSockets.
- **Resolvers**: Functions on the server that fetch the actual data for each field in the schema.

### Key Benefits
- **No Over-fetching**: Clients request only the fields they need.
- **Single Endpoint**: Typically `/graphql`. No need for multiple versioned endpoints.
- **Strongly Typed**: The schema acts as a contract and self-documenting resource.
- **Introspection**: Tools (like GraphiQL) can query the schema itself to discover available data.

## Go Context
In Go, popular libraries for GraphQL include `graphql-go/graphql` (standard-like) and `99designs/gqlgen` (schema-first code generation).

### Example: Simple Schema with `graphql-go`
```go
package main

import (
	"encoding/json"
	"fmt"
	"github.com/graphql-go/graphql"
)

func main() {
	// Schema definition
	fields := graphql.Fields{
		"hello": &graphql.Field{
			Type: graphql.String,
			Resolve: func(p graphql.ResolveParams) (interface{}, error) {
				return "world", nil
			},
		},
	}
	rootQuery := graphql.ObjectConfig{Name: "RootQuery", Fields: fields}
	schemaConfig := graphql.SchemaConfig{Query: graphql.NewObject(rootQuery)}
	schema, _ := graphql.NewSchema(schemaConfig)

	// Query string
	query := "{ hello }"
	params := graphql.Params{Schema: schema, RequestString: query}
	r := graphql.Do(params)
	
	// Print result
	rJSON, _ := json.Marshal(r)
	fmt.Printf("%s\n", rJSON)
}
```

## Interview Questions
- **Q: What are over-fetching and under-fetching?**
- **A:** Over-fetching is when an API returns more data than the client needs (wasting bandwidth). Under-fetching is when a single endpoint doesn't provide enough data, forcing the client to make multiple calls (the N+1 problem). GraphQL solves both by letting the client specify exactly what it needs.

- **Q: What is a resolver in GraphQL?**
- **A:** A resolver is a function that populates the data for a specific field in the schema. It can fetch data from a database, a cache, or another API.

- **Q: How does GraphQL handle versioning?**
- **A:** GraphQL typically avoids versioning by adding new fields and types to the schema without removing old ones. Deprecated fields can be marked with a `@deprecated` directive to warn clients without breaking existing integrations.
