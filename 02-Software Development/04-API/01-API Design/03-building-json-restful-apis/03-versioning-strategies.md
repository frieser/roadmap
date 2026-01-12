---
---
# API Versioning Strategies

API versioning is the process of managing changes to an API in a way that ensures backward compatibility for existing clients while allowing for innovation and breaking changes.

## 1. URI Versioning
The version is included directly in the URL path.
*   **Example**: `https://api.example.com/v1/users`
*   **Pros**:
    *   Explicit and easy to see.
    *   Easy to cache (unique URL per version).
    *   Simple to test in a browser.
*   **Cons**:
    *   Violates the principle that a URI should represent a resource, not its representation.
    *   Can lead to URI "bloat" as versions increase.

## 2. Header Versioning (Content Negotiation)
The version is specified in a request header, typically `Accept` or a custom header.
*   **Example**: `Accept: application/vnd.myapi.v1+json` or `X-API-Version: 1`
*   **Pros**:
    *   Clean, semantic URIs (one URI per resource).
    *   Technically follows REST principles better (Content Negotiation).
*   **Cons**:
    *   Harder to test (requires tools like `curl` or Postman).
    *   Cannot be easily bookmarked or shared via link.
    *   Caching is more complex (must use the `Vary` header).

## 3. Query Parameter Versioning
The version is passed as a query string parameter.
*   **Example**: `https://api.example.com/users?v=1`
*   **Pros**:
    *   Easy to implement.
    *   Easy to test in browsers.
    *   Can default to a specific version if the parameter is missing.
*   **Cons**:
    *   Query parameters are usually for filtering/sorting, not versioning.
    *   Can clutter the URL if other parameters are used.

## 4. Comparison Table

| Strategy | Location | REST Purist? | Testing | Caching |
| :--- | :--- | :--- | :--- | :--- |
| **URI** | Path (`/v1/`) | No | Easy | Simple |
| **Header** | Headers | Yes | Moderate | Complex |
| **Query** | Param (`?v=1`) | No | Easy | Simple |

## 5. Go: Implementation with Gin
In Go, the most common way to implement versioning is through **Routing Groups**.

### URI Versioning (Standard)
```go
package main

import "github.com/gin-gonic/gin"

func main() {
    r := gin.Default()

    // Version 1
    v1 := r.Group("/v1")
    {
        v1.GET("/users", getV1Users)
    }

    // Version 2
    v2 := r.Group("/v2")
    {
        v2.GET("/users", getV2Users)
    }

    r.Run()
}
```

### Header Versioning (Middleware)
You can use middleware to detect the version and route accordingly or set a context value.

```go
func VersionMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        version := c.GetHeader("X-API-Version")
        if version == "" {
            version = "1" // Default
        }
        c.Set("api_version", version)
        c.Next()
    }
}
```

## 6. Interview Questions
1.  **When should you version an API?**
    *   When making breaking changes (removing fields, changing data types, changing endpoint logic). Non-breaking changes (adding fields) usually don't require a new version.
2.  **What is the "Sunset" header?**
    *   An HTTP header used to communicate that an API version is being deprecated and when it will be taken offline.
3.  **How do you handle deprecation?**
    *   Communicate clearly to developers, provide a transition period, use the `Warning` or `Sunset` headers, and eventually return `410 Gone`.
4.  **Why is URI versioning the most popular despite REST principles?**
    *   Pragmatism. It’s visible, easy to debug, and works seamlessly with standard caching infrastructure.
