#API
---
---

# Proper Response Codes

## Summary
Using **Semantic HTTP Status Codes** is essential for both security and developer experience (DX).
*   **Security**: Misleading codes (like returning 200 OK for an error) confuse security scanners and automated monitoring. Using incorrect error codes (e.g., 404 vs 403) can leak information about resource existence.
*   **UX**: Clients rely on specific codes to trigger logic (e.g., 401 triggers a re-login flow, 429 triggers a retry backoff).

## Detailed Explanation

### 1. Common Codes & Nuances
*   **200 OK**: Standard success.
*   **201 Created**: Resource created successfully. Should return a `Location` header.
*   **202 Accepted**: Request received but processing is not complete (Async jobs).
*   **400 Bad Request**: Client sent invalid data (validation error).
*   **401 Unauthorized**: Authentication is missing or invalid. "I don't know who you are."
*   **403 Forbidden**: Authentication is valid, but permissions are lacking. "I know who you are, but you can't do this."
*   **404 Not Found**: Resource doesn't exist.
    *   *Security Note*: Sometimes used instead of 403 to hide the existence of a private resource.
*   **405 Method Not Allowed**: Wrong HTTP verb (e.g., POSTing to a read-only endpoint).
*   **429 Too Many Requests**: Rate limit exceeded.
*   **500 Internal Server Error**: Generic server failure. Ideally, users should rarely see this.

### 2. Information Leakage: 403 vs 404
If a user requests `GET /users/123`:
*   If user 123 exists but the requester isn't an admin -> **403 Forbidden** confirms ID 123 is valid.
*   **Secure Approach**: Return **404 Not Found** for both "ID doesn't exist" and "ID exists but you can't access it." This prevents **ID Enumeration**.

### 3. Go Example: Helper Functions
Standardizing responses prevents "magic numbers" in code.

```go
func RespondJSON(w http.ResponseWriter, status int, payload interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(payload)
}

func RespondError(w http.ResponseWriter, status int, message string) {
    RespondJSON(w, status, map[string]string{"error": message})
}

// Usage
func GetUser(w http.ResponseWriter, r *http.Request) {
    if !authorized {
        // Secure: Don't reveal it exists
        RespondError(w, http.StatusNotFound, "User not found")
        return
    }
    RespondJSON(w, http.StatusOK, user)
}
```

## Interview Questions

### 1. What is the difference between 401 and 403?
**401 (Unauthorized)** means the client has not authenticated yet (or the token is invalid). The fix is to log in. **403 (Forbidden)** means the client *is* authenticated but does not have the necessary permissions (scope/role) to access the resource. The fix is to ask the admin for more access.

### 2. Why is returning "200 OK" with an error body (e.g., GraphQL style) controversial in REST?
It breaks the HTTP protocol semantics. Proxies, caches, and monitoring tools rely on the status code. A "200 OK" implies the request succeeded and can be cached, which is disastrous if the body actually contains an error ("Out of Stock"). It also forces clients to parse the body to know if the request worked.

### 3. When should you return 202 Accepted?
When the request initiates a background process that hasn't finished yet. For example, uploading a video for transcoding. The server accepts the file (202) and likely returns a URL to poll for status, rather than keeping the connection open until the job finishes.

### 4. How can 404 Not Found be used as a security feature?
By returning 404 instead of 403 for unauthorized access to specific resources, you prevent attackers from scanning IDs (Enumeration) to find which resources exist. This is "Security by Obscurity," but acceptable as an additional layer for sensitive IDs.
