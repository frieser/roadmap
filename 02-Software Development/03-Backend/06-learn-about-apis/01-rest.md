---
---

## Summary
Representational State Transfer (REST) is an architectural style for designing networked applications. It relies on a stateless, client-server, cacheable communications protocol — virtually always HTTP. RESTful applications use HTTP requests to post data (create and/or update), read data (make queries), and delete data.

## Detailed Explanation
REST was defined by Roy Fielding in his 2000 PhD dissertation. It is not a protocol or a standard, but a set of architectural constraints.

### Core Constraints
1. **Client-Server Architecture**: Separation of concerns between the user interface and data storage.
2. **Statelessness**: Each request from client to server must contain all the information necessary to understand and complete the request. The server cannot use any saved context on the server.
3. **Cacheability**: Responses must define themselves as cacheable or not to prevent clients from reusing stale or inappropriate data.
4. **Layered System**: A client cannot ordinarily tell whether it is connected directly to the end server or to an intermediary.
5. **Uniform Interface**: This is the most critical constraint. It includes:
    * Identification of resources (URIs).
    * Manipulation of resources through representations.
    * Self-descriptive messages.
    * HATEOAS (Hypermedia as the Engine of Application State).

### HTTP Methods in REST
- `GET`: Retrieve a representation of a resource.
- `POST`: Create a new resource or perform an action.
- `PUT`: Replace a resource or create it if it doesn't exist (Idempotent).
- `PATCH`: Partially update a resource.
- `DELETE`: Remove a resource.

## Go Context
In Go, REST APIs are commonly built using the standard library's `net/http` package or frameworks like Gin, Echo, or Fiber.

### Example: Simple REST API with Gin
```go
package main

import (
	"net/http"
	"github.com/gin-gonic/gin"
)

type User struct {
	ID   string `json:"id"`
	Name string `json:"name"`
}

var users = []User{
	{ID: "1", Name: "Alice"},
}

func main() {
	r := gin.Default()

	r.GET("/users", func(c *gin.Context) {
		c.JSON(http.StatusOK, users)
	})

	r.POST("/users", func(c *gin.Context) {
		var newUser User
		if err := c.ShouldBindJSON(&newUser); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}
		users = append(users, newUser)
		c.JSON(http.StatusCreated, newUser)
	})

	r.Run(":8080")
}
```

## Interview Questions
- **Q: What is the difference between PUT and PATCH?**
- **A:** PUT replaces the entire resource with the provided representation, while PATCH performs a partial update to the resource. PUT is generally expected to be idempotent.

- **Q: What does "statelessness" mean in REST?**
- **A:** It means the server does not store any session state about the client. Every request must contain all necessary information (like authentication tokens) to process it independently.

- **Q: What are idempotent methods?**
- **A:** An idempotent method is one that can be called multiple times without changing the result beyond the initial application. `GET`, `PUT`, and `DELETE` are idempotent, while `POST` is not.
