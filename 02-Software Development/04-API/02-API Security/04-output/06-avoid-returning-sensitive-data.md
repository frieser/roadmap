#API
---
---

# Avoid Returning Sensitive Data

## Summary
One of the most common API vulnerabilities is **Information Leakage**. This occurs when an API returns more information than the client needs or is authorized to see. This includes:
*   **Stack Traces**: Revealing internal code paths and library versions.
*   **Debug Info**: Environment variables, config paths.
*   **PII (Personally Identifiable Information)**: Returning a full User object (with email, phone, SSN) when the UI only requested the display name.

## Detailed Explanation

### 1. Custom Error Handling in Go
By default, `http.Error` or `panic` might print details to the response. In production, you must catch panics and return generic error messages.

**Bad (Leaking Implementation Details):**
```go
func unsafeHandler(w http.ResponseWriter, r *http.Request) {
    db, err := connectDB()
    if err != nil {
        // LEAK: "dial tcp 127.0.0.1:5432: connect: connection refused"
        http.Error(w, err.Error(), 500) 
        return
    }
}
```

**Good (Sanitized):**
```go
func safeHandler(w http.ResponseWriter, r *http.Request) {
    db, err := connectDB()
    if err != nil {
        // Log the real error internally
        log.Printf("DB Error: %v", err)
        // Return a generic safe message
        http.Error(w, "Internal Server Error", 500)
        return
    }
}
```

### 2. JSON Response Filtering
Go structs often map directly to database tables. If you simply marshal the struct, you might expose sensitive columns.

**Solution**: Use separate **DTOs (Data Transfer Objects)** or struct tags.

```go
type User struct {
    ID           int    `json:"id"`
    Username     string `json:"username"`
    PasswordHash string `json:"-"` // Never output this
    Email        string `json:"email,omitempty"` // Only output if necessary
}
```

### 3. Environment Separation (Build Tags)
Use environment variables or build tags to conditionally enable debug routes (like `/debug/pprof`) only in development, ensuring they are completely compiled out or unreachable in production binaries.

## Interview Questions

### 1. What is the risk of returning stack traces to the client?
Stack traces reveal the internal architecture of the application, including file paths, function names, and versions of third-party libraries. Attackers use this "reconnaissance" data to identify known vulnerabilities (CVEs) in specific libraries or to map out the logic flow for SQL injection attacks.

### 2. How can you automate the prevention of sensitive data leakage in Go?
*   Use **linters** (like `gosec`) to detect hardcoded credentials.
*   Use **struct tags** (`json:"-"`) to explicitly exclude sensitive fields.
*   Implement **middleware** that recovers from panics and replaces the body with a static 500 error page/JSON before sending it to the client.

### 3. Why is it dangerous to return a full database model as a JSON response?
It violates the principle of **Least Privilege**. Even if the UI doesn't display the `is_admin` or `salary` field, the data is still present in the JSON payload inspected via browser network tools. An attacker can scrape this hidden data. Always map DB models to specific Response structs (DTOs) that contain *only* the data the client needs.

### 4. What is a "Blind" error message?
A message that acknowledges failure without giving details. For example, on a Login page, returning "Invalid Username or Password" instead of "User not found" prevents attackers from enumerating valid usernames.
