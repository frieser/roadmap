# Slog and Regexp

## Summary
Go 1.21 introduced `log/slog`, a standard library for structured logging (JSON, key-value pairs) that supersedes the basic `log` package for modern applications. The `regexp` package implements regular expression search using the RE2 syntax, guaranteeing linear time complexity `O(n)` performance (avoiding ReDoS attacks) at the cost of some advanced features like backreferences.

## Detailed Explanation

### 1. Structured Logging (`log/slog`)
Traditional logging (`log.Println`) outputs unstructured text, which is hard for machines (Datadog, Splunk) to parse. `slog` outputs key-value pairs structure.

**Basic Usage:**
```go
import "log/slog"

func main() {
    // Default text handler
    slog.Info("User logged in", "user_id", 42, "ip", "192.168.1.1")
    // Output: time=... level=INFO msg="User logged in" user_id=42 ip=192.168.1.1
}
```

**JSON Handler (Production standard):**
```go
import (
    "log/slog"
    "os"
)

func main() {
    logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
    
    logger.Error("Database connection failed", 
        "retry_count", 3,
        "error", "timeout",
    )
    // Output: {"time":"...","level":"ERROR","msg":"Database connection failed","retry_count":3,"error":"timeout"}
}
```

### 2. Regular Expressions (`regexp`)
Go's regex engine is safe for use on untrusted input because it guarantees execution time linear to the input size.

**Compilation:**
Always compile regexes outside loops (usually in `init` or package level `var`) using `MustCompile`.
```go
var emailRegex = regexp.MustCompile(`^[a-z0-9._%+\-]+@[a-z0-9.\-]+\.[a-z]{2,4}$`)
```
*   `Compile`: Returns error if regex is invalid.
*   `MustCompile`: Panics if invalid. Safe for globals.

**Matching:**
```go
if emailRegex.MatchString("alice@example.com") {
    // Valid
}

// Finding submatches (Capture groups)
re := regexp.MustCompile(`(\w+)-(\d+)`)
matches := re.FindStringSubmatch("item-123")
// matches[0] = "item-123" (full match)
// matches[1] = "item"
// matches[2] = "123"
```

## Interview Questions

**Q: Why does Go's `regexp` package not support look-arounds or backreferences?**
**A:** Go uses the RE2 engine, which prioritizes safety and performance guarantees. Features like backreferences require backtracking engines, which can have exponential time complexity `O(2^n)` in worst cases (Catastrophic Backtracking or ReDoS). Go chooses to forbid these features to ensure `O(n)` linear performance, making it safe to run regexes on user inputs.

**Q: What is the difference between `log` and `log/slog`?**
**A:** `log` is a basic logger that writes formatted strings. `slog` (Structured Log) is designed to write structured data (key-value pairs), typically in JSON format. Structured logging allows log aggregation systems to easily index and query fields (e.g., `search where user_id=42`) without complex text parsing rules.

**Q: When should you use `regexp.MustCompile` vs `regexp.Compile`?**
**A:** Use `MustCompile` for global variables or constants where a regex syntax error is a programmer error that should prevent the program from starting. Use `Compile` when the regex pattern comes from user input or runtime configuration, so you can handle the error gracefully without crashing.
