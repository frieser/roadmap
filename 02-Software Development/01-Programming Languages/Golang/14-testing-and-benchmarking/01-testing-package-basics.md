# Testing Package Basics

## Summary
The Go standard library provides a built-in testing framework via the `testing` package. Unlike other languages that require external frameworks (like JUnit or PyTest), Go treats testing as a first-class citizen with a simple, lightweight approach. Tests are placed in files ending with `_test.go`, run alongside the production code, and executed via the `go test` command.

## Detailed Explanation

### The `testing` Package
Go's testing philosophy favors simplicity and standard code over magic assertions. A test is just a function that executes code and checks results using standard `if` statements.

### Key Rules
1.  **File Naming**: Test files must end in `_test.go` (e.g., `calc_test.go`). These files are excluded from regular builds.
2.  **Package Naming**:
    *   Same package: `package math` (allows testing private internals).
    *   Black-box testing: `package math_test` (ensures you only test the public API).
3.  **Function Signature**: Test functions must start with `Test` and take a single argument `t *testing.T`.
    ```go
    func TestAdd(t *testing.T) { ... }
    ```

### Common `testing.T` Methods
*   `t.Error(args...)`: Log error and mark test as failed, but continue execution.
*   `t.Errorf(format, args...)`: Formatted error logging.
*   `t.Fatal(args...)`: Log error and **stop** the test function immediately.
*   `t.Log(args...)`: Log information (visible only if test fails or with `-v` flag).
*   `t.Helper()`: Mark the calling function as a helper (so logs show the caller's line number, not the helper's).

### Code Example

**`calculator.go`**
```go
package calculator

func Add(a, b int) int {
	return a + b
}
```

**`calculator_test.go`**
```go
package calculator

import "testing"

func TestAdd(t *testing.T) {
	result := Add(2, 3)
	expected := 5

	if result != expected {
		t.Errorf("Add(2, 3) = %d; want %d", result, expected)
	}
}
```

### Running Tests
*   `go test`: Run tests in current directory.
*   `go test ./...`: Run tests in all subdirectories.
*   `go test -v`: Verbose output (shows passed tests and logs).
*   `go test -run TestName`: Run a specific test.

## Interview Questions

**Q: What is the difference between `t.Error` and `t.Fatal`?**
**A:** `t.Error` marks the test as failed but continues executing the rest of the test function, which is useful for collecting multiple failures. `t.Fatal` marks the test as failed and stops execution immediately, which is useful when a setup step fails (e.g., database connection) and further assertions would be meaningless.

**Q: Why do we sometimes see `package foo_test` instead of `package foo` in test files?**
**A:** Using `package foo_test` (suffixed with `_test`) enforces "black-box testing". It forces the test code to interact with the package `foo` only via its exported (public) API, just like a real client would. This helps decouple tests from internal implementation details.

**Q: How do you skip a test in Go?**
**A:** You can skip a test at runtime using `t.Skip("reason")`. This is often used for integration tests that require external resources (like a DB) that might not be available, or to skip slow tests during "short" runs (checked via `testing.Short()`).
