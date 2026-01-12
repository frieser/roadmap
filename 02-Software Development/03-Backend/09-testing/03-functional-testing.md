---
---

## Summary
Functional testing, also known as Black-Box testing, verifies that the system performs as expected from an end-user perspective. It focuses on the "What" (requirements) rather than the "How" (implementation), ensuring that given a specific input, the system produces the correct output across the entire stack.

## Detailed Explanation
Functional tests validate the business requirements. In a backend context, this often means testing an entire API endpoint or a complex workflow that spans multiple services.

### Key Characteristics
- **Perspective**: External. The test doesn't know about the internal code structure.
- **Coverage**: End-to-end. It usually involves the full HTTP stack, business logic, and database.
- **Goal**: To ensure the user's needs are met and the features work according to the specification.

### Functional Testing in Go
Go's standard library includes the `net/http/httptest` package, which is excellent for functional testing of web servers.

#### `net/http/httptest`
- **`httptest.NewRecorder()`**: Captures the response from an HTTP handler without needing a live network connection.
- **`httptest.NewServer()`**: Starts a local HTTP server for testing clients or end-to-end flows.

#### BDD (Behavior Driven Development)
Some teams prefer a more descriptive approach using tools like **Godog** (Cucumber for Go), which allows writing tests in plain English (Gherkin syntax).

## Go-specific Examples

### Functional Test for an API Endpoint
```go
package handlers_test

import (
    "net/http"
    "net/http/httptest"
    "testing"
    "strings"
    "github.com/stretchr/testify/assert"
    "myproject/internal/api"
)

func TestCreateUser_Functional(t *testing.T) {
    // Setup the actual router and application
    router := api.SetupRouter()

    // Define the request payload
    payload := `{"name": "Alice", "email": "alice@example.com"}`
    req, _ := http.NewRequest("POST", "/users", strings.NewReader(payload))
    req.Header.Set("Content-Type", "application/json")

    // Record the response
    w := httptest.NewRecorder()
    router.ServeHTTP(w, req)

    // Assertions
    assert.Equal(t, http.StatusCreated, w.Code)
    assert.Contains(t, w.Body.String(), `"id":`)
}
```

### Using `httptest.Server` for E2E
```go
func TestExternalIntegration(t *testing.T) {
    // Mock an external service that our app calls
    ts := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
        w.Write([]byte(`{"status": "ok"}`))
    }))
    defer ts.Close()

    // Configure our app to point to ts.URL
    app := myapp.New(myapp.Config{ExternalAPI: ts.URL})
    
    res := app.DoSomething()
    assert.True(t, res.Success)
}
```

## Interview Questions
**Q: What is the main difference between Integration Testing and Functional Testing?**
**A:** Integration testing focuses on the technical interaction between components (e.g., "Does the service talk to the DB?"). Functional testing focuses on the business outcome (e.g., "When I register a user, do they get a confirmation email and a 201 Created status?"). Functional tests are often broader in scope.

**Q: How does `httptest.NewRecorder()` help in testing?**
**A:** It implements the `http.ResponseWriter` interface, allowing you to pass it into your HTTP handlers. You can then inspect the status code, headers, and body that the handler wrote, making it easy to test web logic without a live server.

**Q: Why is it important to use the actual router in functional tests?**
**A:** Using the actual router (e.g., Gin, Chi, or StdLib Mux) ensures that your tests cover routing logic, middleware (auth, logging, recovery), and parameter parsing, providing a more realistic validation of the API.
