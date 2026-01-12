---
tags: [security, api, logging, go]
---

# Ensure No Sensitive Data Logging

## Summary
Sensitive data logging occurs when Personal Identifiable Information (PII) or sensitive authentication data (credentials, tokens) is inadvertently recorded in system logs. This can lead to compliance violations (GDPR, PCI-DSS) and security breaches if logs are accessed by unauthorized parties or stored in less secure environments. Preventing this requires proactive masking, redaction, and tokenization at the application level.

## Detailed Explanation

### PII and PCI-DSS Compliance
*   **PII (Personally Identifiable Information):** Any data that could potentially identify a specific individual (Email, Phone, SSN, Home Address).
*   **PCI-DSS (Payment Card Industry Data Security Standard):** Strict requirements for handling cardholder data. Requirement 3 specifically mandates the protection of stored cardholder data, and logging "Sensitive Authentication Data" (like CVV or full magnetic stripe data) is strictly prohibited.

### Common Pitfalls
1.  **Request/Response Dumps:** Logging entire HTTP request or response bodies often captures passwords, credit card numbers, or session tokens.
2.  **Authorization Headers:** Logging `Authorization: Bearer <token>` or `Cookie` headers.
3.  **Exception Stack Traces:** Some frameworks dump the state of local variables during a crash, which may contain sensitive values.
4.  **Query Parameters:** Passing API keys or tokens in URLs (e.g., `?api_key=secret`) which are logged by default by web servers (Nginx, Apache).

### Protection Techniques
*   **Data Masking:** Partial hiding of data (e.g., `4532-XXXX-XXXX-1234`). Useful for debugging without exposing the full secret.
*   **Data Redaction:** Replacing sensitive values with a static placeholder like `[REDACTED]` or `***`.
*   **Tokenization:** Replacing sensitive data with a non-reversible token. The mapping is stored in a highly secure, isolated "vault."
*   **Log Scrubbing:** Using middleware or log handlers to intercept and modify log entries before they are written to disk or sent to a log aggregator.

---

## Go (Golang) Application

In Go, the modern way to handle this is using `log/slog` and its `ReplaceAttr` function or by implementing a custom `Handler`.

### Example: Log Scrubbing with slog

```go
package main

import (
	"log/slog"
	"os"
	"strings"
)

// List of sensitive keys to redact
var sensitiveKeys = map[string]bool{
	"password":   true,
	"secret":     true,
	"token":      true,
	"cvv":        true,
	"auth_token": true,
}

func main() {
	// Create a JSON handler that redacts sensitive attributes
	opts := &slog.HandlerOptions{
		ReplaceAttr: func(groups []string, a slog.Attr) slog.Attr {
			// Check if the attribute key is in our sensitive list
			if _, ok := sensitiveKeys[strings.ToLower(a.Key)]; ok {
				return slog.String(a.Key, "[REDACTED]")
			}
			return a
		},
	}

	logger := slog.New(slog.NewJSONHandler(os.Stdout, opts))
	slog.SetDefault(logger)

	// Example logs
	slog.Info("user login attempt", "user", "john_doe", "password", "secret123")
	slog.Info("payment processing", "amount", 100, "cvv", "123")
	slog.Info("api request", "method", "GET", "auth_token", "abc-123-def")
}
```

### Middleware Integration (Gin Example)
When using a web framework like Gin, you should redact sensitive fields from the request body before logging.

```go
func RedactMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		// Before logging the request, you might want to scrub specific headers
		headers := c.Request.Header
		if headers.Get("Authorization") != "" {
			// Do not log the actual token
		}
		c.Next()
	}
}
```

---

## Interview Questions

**Q: How do you prevent sensitive data from being logged in a production environment?**
**A:** Implementation of multiple layers: Use log handlers with redaction logic (like `ReplaceAttr` in Go), avoid logging entire request/response bodies, use environment-specific log levels, and implement "scrubbing" at the log aggregator level (e.g., ELK/Datadog filters) as a fallback.

**Q: What is the difference between data masking and data redaction?**
**A:** Redaction completely removes or replaces the data with a static placeholder (e.g., `[REDACTED]`), making it impossible to see any part of the original value. Masking partially hides the data (e.g., `XXXX-1234`) to preserve some utility for support or debugging without exposing the full sensitive value.

**Q: Why is it dangerous to log JWTs?**
**A:** While JWTs are encoded, they are not encrypted by default. Anyone with access to the logs can decode the payload (e.g., via jwt.io) to see user IDs, roles, and emails. If the secret key is leaked, they can also be used for session hijacking.

**Q: If you must log a request body for debugging, how do you do it safely?**
**A:** Use a middleware that parses the body, iterates through the keys, and applies a redaction/masking function to a whitelist/blacklist of fields before passing the "clean" map to the logger. NEVER log the raw byte stream if it contains PII.
