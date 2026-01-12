---
---

## Summary
API security refers to the practices and tools used to protect APIs from attacks, ensure data privacy, and maintain service availability. APIs are often a prime target because they provide direct access to data and backend logic.

## Detailed Explanation
### Key Best Practices
- **Use HTTPS Everywhere**: Never transmit data over plain text HTTP.
- **Authentication & Authorization**: Use robust standards like OAuth2 and OpenID Connect.
- **Rate Limiting & Throttling**: Protect against DoS attacks and brute-forcing by limiting how many requests a client can make.
- **Input Validation**: Never trust client data. Validate and sanitize every piece of input.
- **Output Sanitization**: Don't leak internal implementation details (like stack traces) in error responses.
- **API Versioning**: Allows you to phase out old, potentially insecure endpoints without breaking existing clients.

### API Gateways
Many organizations use an API Gateway (like Kong, AWS API Gateway, or Nginx) to centralize security concerns like authentication, rate limiting, and logging.

## Go Context
Go's standard library and middleware ecosystem make it easy to implement these practices.

### Example: Simple Rate Limiting in Go
```go
import "golang.org/x/time/rate"

var limiter = rate.NewLimiter(1, 3) // 1 request per second, burst of 3

func limitMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if !limiter.Allow() {
            http.Error(w, "Too Many Requests", http.StatusTooManyRequests)
            return
        }
        next.ServeHTTP(w, r)
    })
}
```

## Interview Questions
- **Q: What is the risk of returning detailed error messages in an API?**
- **A:** Detailed error messages (like database stack traces) can reveal internal implementation details, such as table names, database versions, or file paths, which an attacker can use to plan a more targeted attack.

- **Q: How does Rate Limiting protect an API?**
- **A:** It prevents individual clients from overwhelming the server with too many requests, whether accidentally or as part of a Denial of Service (DoS) attack. It also makes brute-force attacks against authentication endpoints much slower and more difficult.

- **Q: What is a "JWT Scopes" or "Claims-based" authorization?**
- **A:** It is a granular way of granting permissions. Instead of just saying a user is "authenticated," the token contains specific "scopes" (e.g., `read:users`, `write:orders`) that define exactly what actions the bearer is allowed to perform.
