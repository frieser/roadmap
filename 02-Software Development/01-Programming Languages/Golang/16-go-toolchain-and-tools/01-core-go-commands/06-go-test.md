# Go Test Command

## Summary
`go test` is the unified command for running tests, benchmarks, and examples. It automatically finds files ending in `*_test.go`, compiles them along with the package under test, and executes the test functions. It supports filtering, caching, race detection, and coverage analysis.

## Detailed Explanation

### Common Flags

1.  **`-v` (Verbose)**: Prints the name of each test as it runs and its status (PASS/FAIL).
    ```bash
    go test -v ./...
    ```

2.  **`-run <regex>`**: Run only tests matching the regex.
    ```bash
    go test -run TestAuth/Login  # Run specific subtest
    ```

3.  **`-cover`**: Enable coverage analysis.
    ```bash
    go test -coverprofile=c.out
    ```

4.  **`-race`**: Enable the race detector. Crucial for concurrent code.
    ```bash
    go test -race ./...
    ```

5.  **`-short`**: Tells tests to skip long-running tests. (Requires tests to check `if testing.Short() { t.Skip() }`).

6.  **`-count=1`**: Disable test caching. Forces tests to re-run even if the code hasn't changed.

### Test Caching
Go caches successful test results. If you run `go test` twice without changing code, the second run will be instant and output `(cached)`. This speeds up CI significantly. Use `go clean -testcache` to clear it.

## Interview Questions

**Q: How do you run tests for the current package and all subpackages?**
**A:** Use the wildcard pattern `./...` (dot-slash-dot-dot-dot).
```bash
go test ./...
```
This tells the Go tool to traverse the directory tree recursively.

**Q: What does `go test -race` do and why is it important?**
**A:** It compiles the test binary with the **Data Race Detector** enabled. It monitors memory access at runtime and crashes the test if two goroutines access the same variable concurrently (where at least one access is a write). It is essential because race conditions are notoriously hard to debug and often don't cause failures during normal execution but cause data corruption in production.

**Q: How can you disable test caching?**
**A:** The idiomatic way is `go test -count=1`. Since `go test` caches results based on the inputs (source code, environment, flags), forcing the test to run "1 time" explicitly bypasses the cache logic which typically defaults to "run once if not cached".
