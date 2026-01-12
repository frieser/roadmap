---
---

## Summary
**GraphQL** is a query language for APIs and a runtime for fulfilling those queries. Unlike REST (multiple endpoints), GraphQL exposes a **single endpoint**. Clients specify exactly what data they need, preventing Over-fetching and Under-fetching.

## Detailed Explanation
### Key Concepts
1.  **Schema**: Strongly typed definition of data (`type User { id: ID! name: String }`).
2.  **Query**: Reading data.
3.  **Mutation**: Writing data.
4.  **Resolver**: Function that fetches the data for a specific field.

### REST vs GraphQL
*   **Over-fetching**: REST returns the whole User object when you only need the name. GraphQL returns only `name`.
*   **Under-fetching**: REST requires 3 requests to get User + Posts + Comments. GraphQL gets it all in 1 nested query.

### Go Context
Popular library: `gqlgen` (Schema-first approach).

```graphql
# Schema
type Query {
    user(id: ID!): User
}

type User {
    id: ID!
    name: String!
    todos: [Todo!]!
}
```

## Interview Questions
**Q: What is the N+1 problem in GraphQL?**
A: If you query a list of 10 users and ask for their "posts", a naive resolver might run 1 SQL query for users, then 10 separate SQL queries for posts (11 total). Solution: **DataLoaders** (batching IDs and fetching all posts in 1 query).

**Q: Can you cache GraphQL queries like REST?**
A: Harder. Since GraphQL uses POST and a single endpoint, standard HTTP caching (CDN/Browser) doesn't work well. You need application-level caching (Persistent Queries or normalizing cache on client like Apollo Client).

## Diagram
```mermaid
graph TD
    Client[Client Request] -->|Query: { user { name, posts { title } } }| GQL[GraphQL Server]
    
    GQL -->|Resolver 1| DB1[User DB]
    GQL -->|Resolver 2| DB2[Post DB]
    
    DB1 --> GQL
    DB2 --> GQL
    
    GQL -->|JSON: { user: { name: 'A', posts: [...] } }| Client
```
