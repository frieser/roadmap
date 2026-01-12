# Build Constraints and Tags

## Summary
Build constraints (or build tags) determine which files are included in a package during the build process. They are used for platform-specific code (Windows vs Linux) or for separating integration tests from unit tests.

## Detailed Explanation

### Syntax (Go 1.17+)
Go uses the `//go:build` directive at the top of the file.
*   **Directive**: `//go:build constraint`
*   **Logic**:
    *   `linux && amd64` (AND)
    *   `linux || windows` (OR)
    *   `!cgo` (NOT)

### Filename Suffixes
The `go` tool automatically recognizes files ending in `_$GOOS.go` or `_$GOARCH.go`.
*   `file_linux.go` -> Only builds on Linux.
*   `file_windows_amd64.go` -> Only builds on Windows AMD64.

### Integration Tests
A common pattern is to tag slow tests so they don't run by default.

**File: `db_integration_test.go`**
```go
//go:build integration

package db_test
// ...
```

**Command**:
```bash
go test ./... -tags=integration
```

### Code Example: Conditional Compilation

**File: `main_linux.go`**
```go
package main
import "fmt"
func logOS() { fmt.Println("Running on Linux") }
```

**File: `main_windows.go`**
```go
package main
import "fmt"
func logOS() { fmt.Println("Running on Windows") }
```

**File: `main.go`**
```go
package main
func main() {
    logOS()
}
```

## Interview Questions

**Q: What is the difference between `//go:build` and `// +build`?**
**A:** `//go:build` is the modern syntax introduced in Go 1.17. It supports boolean logic (`&&`, `||`). `// +build` is the legacy syntax.

**Q: How do you exclude a file from the build?**
**A:** Add `//go:build ignore` at the top.

**Q: How do you run tests that have a specific build tag?**
**A:** Use the `-tags` flag: `go test -tags=mytag ./...`.
