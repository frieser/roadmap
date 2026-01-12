---
---

## Summary
GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data. It provides a complete and understandable description of the data in your API, allowing clients to request exactly what they need and nothing more. This eliminates over-fetching and under-fetching while enabling powerful developer tools and schema evolution without versioning.

## Detailed Explanation

### Core Concepts
- **Single Endpoint**: Unlike REST, which exposes multiple resource-specific endpoints (e.g., `/users`, `/posts`), GraphQL typically uses a single HTTP endpoint (usually `/graphql`) to handle all requests.
- **Schema & SDL**: The Schema is the "contract" between the client and server. It is written in **Schema Definition Language (SDL)**.
- **Queries**: Used for read-only data fetching.
- **Mutations**: Used for creating, updating, or deleting data (analogous to POST/PUT/DELETE in REST).
- **Resolvers**: Server-side functions responsible for fetching the data for a specific field in the schema.

### Benefits
- **Efficiency**: Prevents **Over-fetching** (getting more data than needed) and **Under-fetching** (not getting enough data, requiring multiple requests).
- **Strong Typing**: The schema provides a clear contract and enables automatic documentation and validation.
- **Introspection**: The API can be queried to discover its own schema, powering tools like GraphiQL or Apollo Studio.
- **Versioning**: GraphQL reduces the need for API versioning (v1, v2) by allowing fields to be deprecated and new ones added without breaking changes.

### Drawbacks & Challenges
- **Caching Complexity**: Since most GraphQL requests use POST and a single endpoint, standard HTTP-level caching is difficult. Caching must often be handled at the application level or via specialized clients (e.g., Apollo Client).
- **N+1 Problem**: A naive resolver implementation can lead to one database query for a list of items and then N additional queries for each item's nested fields.
- **Query Complexity**: Malicious or poorly designed queries can deeply nest requests, potentially overwhelming the server (mitigated by "Query Depth Limiting" or "Cost Analysis").

### Go Implementation (gqlgen)
In Go, `99designs/gqlgen` is the industry standard for a "schema-first" approach. You define your schema in SDL, and `gqlgen` generates the boilerplate and type-safe interfaces.

#### 1. Schema Definition (`graph/schema.graphql`)
```graphql
type User {
  id: ID!
  name: String!
  email: String!
}

type Query {
  user(id: ID!): User
}

type Mutation {
  createUser(name: String!, email: String!): User!
}
```

#### 2. Resolver Implementation (`graph/schema.resolvers.go`)
```go
package graph

import (
	"context"
	"fmt"
)

type Resolver struct {
    // Add dependencies like database connections here
}

func (r *queryResolver) User(ctx context.Context, id string) (*model.User, error) {
	// Logic to fetch user from DB
	return &model.User{ID: id, Name: "John Doe", Email: "john@example.com"}, nil
}

func (r *mutationResolver) CreateUser(ctx context.Context, name string, email string) (*model.User, error) {
	// Logic to save user to DB
	user := &model.User{ID: "123", Name: name, Email: email}
	return user, nil
}
```

## Interview Questions

**Q: What is the primary difference between GraphQL and REST?**
**A:** REST is resource-oriented, using multiple endpoints and standard HTTP verbs for specific resources. GraphQL is query-oriented, using a single endpoint where the client defines the structure of the returned data.

**Q: What is the N+1 problem in GraphQL and how do you solve it?**
**A:** The N+1 problem occurs when a list of objects is fetched (1 query), and then for each object, a separate query is executed to fetch a nested field (N queries). In Go, this is typically solved using **Dataloaders** (like `graph-gophers/dataloader` or `vikstrous/dataloadgen`), which batch and cache requests.

**Q: How does GraphQL handle error reporting?**
**A:** GraphQL always returns a `200 OK` HTTP status code if the request was successfully parsed and executed. Errors are returned in a top-level `errors` array in the JSON response body, potentially alongside partial data in the `data` field.

**Q: What are Fragments in GraphQL?**
**A:** Fragments are reusable units of a GraphQL query. They allow you to define a set of fields that can be shared across multiple queries, improving maintainability and reducing duplication.

**Q: Explain the concept of "Schema-First" vs "Code-First".**
**A:** **Schema-First** (like `gqlgen`) starts with the SDL file, which is used to generate code. **Code-First** (like `graphql-go`) involves writing the schema using language-specific types and objects, which then defines the API.
