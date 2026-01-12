# Echo Web Framework

## Summary
Echo is a high-performance, extensible, and minimalist web framework for Go. It is known for its optimized HTTP router (Radix tree-based), data binding, and extensive middleware support. It is often cited as a cleaner, slightly more "standard" alternative to Gin, with a focus on simplicity and speed.

## Detailed Explanation

### 1. Basic Structure
Echo follows a similar pattern to other micro-frameworks but encapsulates everything in the `echo.Context`.

```go
package main

import (
	"net/http"
	"github.com/labstack/echo/v4"
)

func main() {
	e := echo.New()
	
	e.GET("/", func(c echo.Context) error {
		return c.String(http.StatusOK, "Hello, World!")
	})
	
	e.Logger.Fatal(e.Start(":1323"))
}
```

### 2. Handler Signature
Echo handlers return an `error`, which simplifies error handling logic compared to standard `http.Handler` (which returns nothing).
```go
func getUser(c echo.Context) error {
  id := c.Param("id")
  return c.JSON(http.StatusOK, map[string]string{"id": id})
}
```

### 3. Key Features
*   **Binding**: Automatically binds request data (JSON, XML, Form) to structs.
    ```go
    u := new(User)
    if err := c.Bind(u); err != nil { return err }
    ```
*   **Middleware**: robust ecosystem (Logger, Recover, CORS, JWT).
*   **HTTP/2**: Automatic support via Go's standard library integration.

## Interview Questions

**Q: What is the main difference between Echo's handler signature and the standard library's?**
**A:** Standard handlers are `func(w http.ResponseWriter, r *http.Request)`. Echo handlers are `func(c echo.Context) error`. Returning an error allows Echo to use a centralized Error Handler middleware to format responses (e.g., as JSON error objects) consistently across the app, rather than manually writing error responses in every handler.

**Q: How does Echo handle routing?**
**A:** Like Gin, Echo uses a high-performance Radix tree-based router. It supports dynamic path parameters (`/users/:id`), match-any wildcards (`/files/*`), and prioritizes static routes over dynamic ones for speed.

**Q: Can you run Echo on a standard `net/http` server?**
**A:** Yes. `echo.Echo` implements the `http.Handler` interface (via `ServeHTTP`). This means it can be used with standard Go tools, or even mounted as a sub-handler within another Go application.
