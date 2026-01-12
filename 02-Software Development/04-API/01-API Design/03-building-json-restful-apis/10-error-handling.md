#API
---
---

## Summary
Proper error handling in APIs is crucial for developer experience (DX) and debugging. It involves returning standard HTTP status codes, consistent JSON error payloads, and not exposing sensitive internal details (like stack traces or SQL queries) to the client.

## Detailed Explanation

### 1. HTTP Status Codes
Use the right code for the right situation. Don't return `200 OK` with `{"error": "failed"}`.
*   **2xx Success**: `200 OK`, `201 Created`, `204 No Content`.
*   **4xx Client Error**:
    *   `400 Bad Request`: Validation failed.
    *   `401 Unauthorized`: Authentication missing/invalid.
    *   `403 Forbidden`: Authenticated, but no permission.
    *   `404 Not Found`: Resource doesn't exist.
    *   `429 Too Many Requests`: Rate limit hit.
*   **5xx Server Error**: `500 Internal Server Error` (Bug/Crash).

### 2. Consistent Error Payload
Define a standard structure and stick to it.
```json
{
  "error": {
    "code": "INVALID_EMAIL",
    "message": "The email format is incorrect.",
    "details": "field 'email' must contain @"
  }
}
```

### 3. Security
**NEVER** return raw internal errors to the client in production.
*   Bad: `{"error": "SQL syntax error near 'WHERE id=1'..."}` (Vulnerable to SQL Injection info gathering).
*   Good: `{"error": "Internal Server Error", "request_id": "req-123"}` (Log the real error internally with the request_id).

## Go-Specific Context/Examples

Go doesn't have exceptions; it treats errors as values. For APIs, we usually create a helper function to format these errors as JSON.

### Example: Custom Error Response Helper

```go
package main

import (
	"encoding/json"
	"log"
	"net/http"
)

// Standard JSON Error response
type APIError struct {
	Code    int    `json:"code"`    // HTTP Status
	Message string `json:"message"` // User-friendly message
	Details string `json:"details,omitempty"` // Optional debugging info
}

func JSONError(w http.ResponseWriter, status int, message string, details string) {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(status)
	
	errResponse := APIError{
		Code:    status,
		Message: message,
		Details: details,
	}
	
	json.NewEncoder(w).Encode(errResponse)
}

func handler(w http.ResponseWriter, r *http.Request) {
	// Simulate an internal error
	err := doDatabaseWork()
	if err != nil {
		// Log the REAL error internally
		log.Printf("Internal DB Error: %v", err)
		
		// Return a generic error to client
		JSONError(w, http.StatusInternalServerError, "Something went wrong", "")
		return
	}
}

func doDatabaseWork() error {
	return nil // or fmt.Errorf("connection failed")
}

func main() {
	http.HandleFunc("/", handler)
	http.ListenAndServe(":8080", nil)
}
```

## Interview Questions

**Q: Why is returning 200 OK for an error (e.g., GraphQL style) controversial in REST?**
**A:** In REST, the HTTP status code is part of the API contract. Monitoring tools, caches, and proxies rely on status codes (e.g., caching 200s, retrying 500s). Returning 200 for errors breaks this semantic layer, forcing every client to parse the body to know if the request actually succeeded.

**Q: How do you handle validation errors for multiple fields?**
**A:** Return a 400 Bad Request with a structured list of errors.
```json
{
  "code": 400,
  "message": "Validation Failed",
  "errors": [
    {"field": "email", "message": "Invalid format"},
    {"field": "age", "message": "Must be > 18"}
  ]
}
```

**Q: What is the "Problem Details for HTTP APIs" (RFC 7807)?**
**A:** It is a standardized JSON format for API errors (media type `application/problem+json`) to avoid everyone inventing their own error structure. It defines fields like `type` (URI to doc), `title`, `status`, `detail`, and `instance`.
