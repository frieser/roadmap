#API
---
---

# Use Proper HTTP Methods

## Summary
Using proper HTTP methods is a fundamental aspect of RESTful API security and design. It aligns the **intent** of the client request with the **behavior** of the server.
*   **Semantic Meaning**: Each method (GET, POST, PUT, DELETE) conveys a specific action.
*   **Safety**: Safe methods (GET, HEAD) should never modify state.
*   **Idempotency**: Idempotent methods (PUT, DELETE) can be repeated multiple times without changing the result beyond the initial application.

## Detailed Explanation

### 1. The HTTP Verbs
*   **GET**: Retrieve a resource. **Safe** and **Idempotent**. Should never trigger actions like "Delete User" or "Purchase Item".
*   **POST**: Create a resource or trigger a process. **Non-Idempotent**. Repeating a POST often results in duplicate records (e.g., duplicate charges).
*   **PUT**: Replace a resource entirely. **Idempotent**. `PUT /users/1` with `{name: "A"}` should result in the same state regardless of how many times it's called.
*   **PATCH**: Partially update a resource. **Non-Idempotent** (technically), though often implemented idempotently.
*   **DELETE**: Remove a resource. **Idempotent**. Deleting a user once returns 200/204; deleting them again returns 404, but the server state (user is gone) remains consistent.

### 2. Security Implications: The CSRF Risk
One of the biggest security risks of misusing HTTP methods is **Cross-Site Request Forgery (CSRF)**.
*   **Scenario**: An API uses `GET /api/delete?id=123` to delete a user.
*   **Attack**: An attacker embeds `<img src="https://api.example.com/delete?id=123">` in an email or website.
*   **Result**: When the victim loads the image, the browser sends the GET request automatically (including cookies), and the user is deleted.
*   **Fix**: Browsers do not automatically perform POST/PUT/DELETE requests for images or links. State-changing actions MUST use non-safe methods (POST/DELETE), which allows Anti-CSRF tokens to be enforced.

### 3. Go Implementation
In Go, you enforce methods using the `http.Handler` or frameworks like Gin/Chi.

```go
func UserHandler(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
        // Retrieve logic
    case http.MethodPost:
        // Create logic
    default:
        http.Error(w, "Method Not Allowed", http.StatusMethodNotAllowed)
    }
}
```

---

## Interview Questions

### 1. What is the difference between "Safe" and "Idempotent" methods?
*   **Safe**: The operation is read-only. It does not alter the server state (e.g., GET).
*   **Idempotent**: The operation can be performed multiple times, and the end state of the server will be the same as if it were performed once (e.g., PUT, DELETE). All safe methods are idempotent, but not all idempotent methods are safe.

### 2. Why shouldn't you use GET for state-changing operations?
Using GET for actions (like creating or deleting) makes the application vulnerable to **CSRF** attacks because browsers fetch GET requests automatically (e.g., image loading, link pre-fetching). It also causes sensitive parameters to be leaked in browser history and server access logs via the URL.

### 3. What status code should you return if a client uses the wrong method?
**405 Method Not Allowed**. Additionally, the server should ideally send an `Allow` header listing the supported methods (e.g., `Allow: GET, POST`).

### 4. Is POST idempotent?
No. By definition, POST is not idempotent. Sending the same POST request twice (e.g., `POST /orders`) typically creates two distinct resources (two orders).

### 5. When would you choose PUT over PATCH?
Use **PUT** when the client is sending the **complete** representation of the resource to replace the existing one. Use **PATCH** when the client is sending a partial set of instructions or fields to update (delta).
