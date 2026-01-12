# Gorilla Mux

## Summary
Gorilla Mux (`gorilla/mux`) was one of the most powerful and popular HTTP routers for Go. It extended the standard `net/http` capabilities with URL parameters (e.g., `/user/{id}`), regular expression matching, and subrouters.
**Note**: As of late 2022, the Gorilla toolkit was archived (read-only mode), though it has since been picked up by new maintainers. However, with Go 1.22's enhanced `http.ServeMux` (which now supports methods and wildcards), the need for Gorilla Mux has diminished for new projects.

## Detailed Explanation

### 1. Variables in Routes
The killer feature of Mux was easy extraction of URL parameters.

```go
r := mux.NewRouter()
r.HandleFunc("/product/{key}", ProductHandler)
r.HandleFunc("/articles/{category}/{id:[0-9]+}", ArticleHandler) // Regex validation

func ArticleHandler(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    category := vars["category"]
    id := vars["id"]
    fmt.Fprintf(w, "Category: %s, ID: %s", category, id)
}
```

### 2. Matching Features
Mux allows matching requests based on complex criteria beyond just the path.
```go
// Match only GET requests
r.HandleFunc("/api", ApiHandler).Methods("GET")

// Match specific host/subdomain
r.Host("www.example.com")

// Match query parameters (e.g., /search?q=golang)
r.HandleFunc("/search", SearchHandler).Queries("q", "{query}")
```

### 3. Middleware
It follows the standard `http.Handler` middleware pattern but provides a `Use()` method for global middleware.
```go
r.Use(loggingMiddleware)
```

## Interview Questions

**Q: With Go 1.22+, do we still need Gorilla Mux?**
**A:** For many standard use cases, no. Go 1.22 introduced method matching (`mux.HandleFunc("POST /items", ...)` ) and wildcards (`/items/{id}`) into the standard library. However, Gorilla Mux is still useful for legacy codebases or if you need complex regex matching in routes (e.g., `{id:[a-z]+}`) which the standard library still doesn't support as flexibly.

**Q: How does `mux.Vars(r)` work internally?**
**A:** It retrieves the route variables from the request's **Context**. When Mux matches a route, it parses the path parameters and stores them in a map within the `http.Request` context, which `mux.Vars` then extracts.

**Q: What happens if two routes match the same URL?**
**A:** Gorilla Mux matches routes in the order they were defined. The first route that matches the request (path, method, headers, etc.) wins. This is why specific routes (e.g., `/products/new`) should be defined *before* generic wildcard routes (e.g., `/products/{id}`).
