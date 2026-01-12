# Zap (Uber)

## Summary
Zap is a blazing fast, structured, leveled logging library developed by Uber. It is designed for high-performance scenarios where logging is hot-path critical. It offers two modes: `Logger` (extremely fast, strongly typed, verbose) and `SugaredLogger` (slightly slower, easier interface, `printf` style).

## Detailed Explanation

### 1. Logger vs SugaredLogger

*   **Zap Logger**: Maximum speed, zero allocation. Usage is verbose.
    ```go
    logger, _ := zap.NewProduction()
    logger.Info("failed to fetch URL",
        zap.String("url", "http://example.com"),
        zap.Int("attempt", 3),
        zap.Duration("backoff", time.Second),
    )
    ```

*   **SugaredLogger**: More convenient, accepts `interface{}`, supports `printf`. 4-10x slower than Logger but still faster than other libraries.
    ```go
    sugar := logger.Sugar()
    sugar.Infow("failed to fetch URL",
        "url", "http://example.com",
        "attempt", 3,
    )
    sugar.Infof("failed to fetch URL: %s", url)
    ```

### 2. Configuration
Zap makes it easy to switch between `Development` (human-readable, stack traces on warnings) and `Production` (JSON, stack traces only on errors).
```go
config := zap.NewProductionConfig()
config.OutputPaths = []string{"stdout"}
logger, _ := config.Build()
```

## Interview Questions

**Q: When should you use `zap.Logger` over `zap.SugaredLogger`?**
**A:** Use the base `zap.Logger` in the "hot paths" of your application—loops or handlers that are executed thousands of times per second—where every memory allocation counts. Use `SugaredLogger` for initialization, configuration, or less critical code paths where code readability is more important than raw nanosecond performance.

**Q: What is "structured logging"?**
**A:** Structured logging means outputting log entries as structured objects (usually JSON) with key-value pairs, rather than unstructured strings. This allows machines to parse, filter, and analyze logs effectively (e.g., "Find all logs where `user_id` is 42 and `latency` > 500ms").

**Q: How does Zap handle sampling?**
**A:** In high-traffic systems, logging every single error can flood the disk/network. Zap has built-in sampling support (in its production config) which can throttle the logging of duplicate entries (e.g., "Log the first 100 entries per second, then only every 100th entry") to maintain system stability under load.
