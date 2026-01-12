# `httptest` for HTTP Tests

## Summary
The `net/http/httptest` package is a hidden gem in Go's standard library. It allows you to test your HTTP handlers and clients without spinning up a real network server or making actual external API calls. It provides two main tools: `ResponseRecorder` (for testing handlers) and `Server` (for testing clients).

## Detailed Explanation

### 1. Testing Handlers with `ResponseRecorder`
When you want to test an HTTP handler (controller), you don't need a running server. A handler is just a function: `func(w http.ResponseWriter, r *http.Request)`.
*   **Input**: Create a dummy request with `httptest.NewRequest`.
*   **Output**: Capture the response with `httptest.NewRecorder` (which implements `http.ResponseWriter`).

#### Code Example: Testing a Handler
```go
func HealthCheckHandler(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    w.Write([]byte("OK"))
}

func TestHealthCheckHandler(t *testing.T) {
    // 1. Create a request
    req := httptest.NewRequest("GET", "/health", nil)

    // 2. Create a ResponseRecorder (which satisfies http.ResponseWriter)
    rr := httptest.NewRecorder()

    // 3. Call the handler directly
    HealthCheckHandler(rr, req)

    // 4. Assert status code
    if status := rr.Code; status != http.StatusOK {
        t.Errorf("handler returned wrong status code: got %v want %v",
            status, http.StatusOK)
    }

    // 5. Assert body
    expected := "OK"
    if rr.Body.String() != expected {
        t.Errorf("handler returned unexpected body: got %v want %v",
            rr.Body.String(), expected)
    }
}
```

### 2. Testing Clients with `httptest.Server`
When testing code that *makes* HTTP calls (an API client), you need a server to talk to. `httptest.NewServer` spins up a real local HTTP server on a random port that closes when the test ends.

#### Code Example: Testing a Client
```go
func TestFetchUserData(t *testing.T) {
    // 1. Start a local server that mocks the external API
    server := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
        w.Write([]byte(`{"name": "Alice"}`))
    }))
    defer server.Close() // Clean up

    // 2. Configure your client to point to this test server
    // server.URL is something like "http://127.0.0.1:54321"
    client := NewClient(server.URL)

    // 3. Run the code under test
    user, err := client.FetchUser()

    // 4. Assertions
    if err != nil {
        t.Fatalf("expected no error, got %v", err)
    }
    if user.Name != "Alice" {
        t.Errorf("expected Alice, got %s", user.Name)
    }
}
```

## Interview Questions

**Q: How do you test an HTTP handler without opening a network port?**
**A:** By using `httptest.NewRecorder()`. It implements the `http.ResponseWriter` interface, capturing the status code, headers, and body written by the handler in memory, allowing you to inspect the results synchronously.

**Q: When would you use `httptest.NewServer`?**
**A:** You use it when you are testing an **HTTP Client** (code that makes outgoing requests). It spins up a real local server that returns mock responses, allowing you to verify that your client parses responses and handles errors correctly without hitting a real external API.

**Q: Can `httptest` handle HTTPS?**
**A:** Yes, via `httptest.NewTLSServer()`. It starts a server using a self-signed certificate. Your client must be configured to trust this certificate (or skip verification) for the connection to succeed.
