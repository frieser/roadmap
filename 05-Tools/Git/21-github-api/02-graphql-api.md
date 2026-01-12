# GitHub GraphQL API

## Summary
The GraphQL API allows you to request exactly the data you need in a single request, avoiding the over-fetching and under-fetching problems of REST.

## Detailed Explanation

### Endpoint
`https://api.github.com/graphql` (POST only).

### Example Query
```graphql
query {
  viewer {
    login
    repositories(first: 3) {
      nodes {
        name
      }
    }
  }
}
```

### Go-specific Context
You can use `shurcooL/githubv4` library in Go to interact with the GraphQL API in a type-safe way.

## Interview Questions
**Q: How are rate limits calculated in GraphQL?**
**A:** Based on "points" (node complexity) rather than request count. A complex query costs more points.
