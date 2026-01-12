#API
---
---

## Summary
GraphQL is a powerful query language for APIs and a runtime for fulfilling those queries with your existing data. Unlike REST, which uses multiple endpoints for different resources, GraphQL typically exposes a single endpoint and allows clients to request exactly the data they need, no more and no less. It was developed by Facebook in 2012 and open-sourced in 2015, becoming a standard for efficient, flexible, and developer-friendly API communication.

## Detailed Explanation

### 1. Core Concepts
- **Schema**: The "contract" between the client and server. Written in Schema Definition Language (SDL), it defines all available types, fields, queries, and mutations.
- **Query**: The operation used by clients to fetch data. It mirrors the shape of the resulting JSON.
- **Mutation**: The operation used to modify data on the server (Create, Update, Delete).
- **Resolver**: A function on the server responsible for fetching the data for a specific field in the schema. Resolvers can fetch from databases, REST APIs, or other microservices.
- **Subscription**: A way to push real-time data from the server to the client, usually implemented via WebSockets.

### 2. Solving REST Pain Points
- **Over-fetching**: REST often returns fixed data structures, forcing clients to download unnecessary fields. In GraphQL, the client selects only the required fields.
- **Under-fetching**: REST might require multiple requests to different endpoints (e.g., `/users/1` then `/users/1/posts`) to get related data. GraphQL allows fetching nested resources in a single request.

### 3. GraphQL in Go
In the Go ecosystem, there are two primary libraries for implementing GraphQL:

#### gqlgen (Schema-First)
`gqlgen` is the most popular library in Go. It uses a **schema-first** approach: you define your schema in `.graphql` files, and it generates the boilerplate Go code for you.

**Example Setup:**
```go
// graph/model/models_gen.go (Generated)
type Todo struct {
    ID   string `json:"id"`
    Text string `json:"text"`
    Done bool   `json:"done"`
}

// graph/schema.resolvers.go
func (r *queryResolver) Todos(ctx context.Context) ([]*model.Todo, error) {
    return r.DB.FindTodos(), nil
}
```

#### graphql-go (Code-First)
`graphql-go` follows the reference implementation of GraphQL. It uses a **code-first** approach where you define your schema using Go structures directly.

**Example:**
```go
var todoType = graphql.NewObject(graphql.ObjectConfig{
    Name: "Todo",
    Fields: graphql.Fields{
        "id":   &graphql.Field{Type: graphql.String},
        "text": &graphql.Field{Type: graphql.String},
        "done": &graphql.Field{Type: graphql.Boolean},
    },
})
```

### 4. Dataloaders and the N+1 Problem
A common performance issue in GraphQL is the **N+1 problem**, where fetching a list of $N$ items results in $N$ additional queries to fetch a related field (e.g., fetching authors for 10 posts).
**Solution**: **Dataloaders** batch and memoize requests. Instead of 10 individual database queries, the Dataloader collects the IDs and performs 1 single query: `SELECT * FROM authors WHERE id IN (...)`.

## Interview Questions

**Q: What is the difference between REST and GraphQL?**
**A:** REST is architectural-style based on resources and HTTP methods across multiple endpoints. GraphQL is a query language with a single endpoint where the client defines the structure of the response. GraphQL solves over-fetching and under-fetching but adds complexity in caching and rate limiting.

**Q: How do you handle Authentication and Authorization in GraphQL?**
**A:** Authentication is usually handled at the transport layer (e.g., JWT in HTTP headers) before reaching GraphQL. Authorization (checking if a user can see a specific field) is best handled inside **Resolvers** or by a dedicated business logic layer to ensure data security.

**Q: What are "Fragments" in GraphQL?**
**A:** Fragments are reusable units of GraphQL queries. They allow you to define a set of fields once and include them in multiple queries, promoting DRY (Don't Repeat Yourself) principles in client-side code.

**Q: What is a Schema Definition Language (SDL)?**
**A:** SDL is the syntax used to write GraphQL schemas. It is language-agnostic and defines the types, fields, and relationships in the API. Example: `type User { id: ID!, name: String }`.

**Q: How does GraphQL handle versioning?**
**A:** Unlike REST which often uses `/v1/` or `/v2/`, GraphQL encourages **evolution without versioning**. You can add new fields without breaking existing clients, and use the `@deprecated` directive to signal that a field should no longer be used.
