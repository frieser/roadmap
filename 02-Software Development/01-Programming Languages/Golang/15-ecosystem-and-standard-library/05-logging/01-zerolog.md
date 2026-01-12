# Zerolog

## Summary
Zerolog is a high-performance, structured logging library for Go. Its main philosophy is **zero allocation**: it avoids heap allocations for standard logging operations to ensure that logging does not become a bottleneck in high-throughput applications. It outputs JSON by default, making it ideal for modern observability stacks (ELK, Datadog).

## Detailed Explanation

### 1. Zero Allocation
Zerolog achieves speed by passing a pointer to the logger and using a fluent API that builds the log event directly into a byte buffer, avoiding the creation of intermediate interface objects (unlike the standard `log` or `logrus`).

### 2. Usage
```go
package main

import (
    "github.com/rs/zerolog"
    "github.com/rs/zerolog/log"
)

func main() {
    // Default: Unix Time, JSON output
    // Set global level
    zerolog.SetGlobalLevel(zerolog.InfoLevel)

    log.Info().
        Str("service", "payment").
        Int("status", 200).
        Msg("Transaction processed")
    
    // Output: {"level":"info","service":"payment","status":200,"time":1600000000,"message":"Transaction processed"}
}
```

### 3. Contextual Logging
You can derive sub-loggers that carry specific context (like a Request ID) throughout a call chain.
```go
// Create a logger with common fields
subLogger := log.With().Str("component", "module_x").Logger()
subLogger.Info().Msg("starting module")
```

## Interview Questions

**Q: How does Zerolog minimize memory allocations compared to `logrus`?**
**A:** `logrus` accepts `interface{}` for its fields, forcing Go to allocate memory to box values (e.g., converting an `int` to an interface). Zerolog uses a strongly-typed API (`.Int()`, `.Str()`, `.Bool()`) which writes the value directly to a byte buffer, avoiding the interface conversion and heap escape.

**Q: Why is JSON output the default for Zerolog?**
**A:** Structured logging (JSON) is the industry standard for production environments. It allows log aggregation systems (like Splunk or CloudWatch) to index fields automatically, enabling queries like `service="payment" AND status > 500`. Human-readable text logs are harder to query programmatically.

**Q: Can Zerolog print human-readable logs for local development?**
**A:** Yes. You can wrap the output in `zerolog.ConsoleWriter`.
```go
log.Logger = log.Output(zerolog.ConsoleWriter{Out: os.Stderr})
```
This formats the JSON into colored, pretty-printed text, but it is much slower and recommended only for development.
