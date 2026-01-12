# Build Tags

## Summary
Build tags (also known as build constraints) allow you to include or exclude specific files from the build process based on conditions like the operating system (`linux`, `windows`), architecture (`amd64`, `arm`), or custom tags (`integration`, `pro`). They are defined using the `//go:build` directive at the top of the file.

## Detailed Explanation

### Syntax
The `//go:build` directive (introduced in Go 1.17) uses boolean logic.

1.  **OR**: `linux || windows` (Include if OS is Linux OR Windows)
2.  **AND**: `linux && amd64` (Include if OS is Linux AND Arch is AMD64)
3.  **NOT**: `!cgo` (Include if CGO is disabled)

Example:
```go
//go:build (linux || darwin) && !cgo

package mypkg
```

### Legacy Syntax (`// +build`)
Before Go 1.17, the syntax was comment-based:
*   `// +build linux,386` meant AND.
*   `// +build linux windows` meant OR.
You will still see this in older codebases, but `go fmt` now automatically synchronizes them with the new `//go:build` syntax.

### File Suffixes
Go also supports implicit build tags via file naming conventions:
*   `file_linux.go`: Automatically built only on Linux.
*   `file_windows_amd64.go`: Built only on Windows AMD64.
*   `file_test.go`: Built only during `go test`.

### Use Case: Integration Tests
You can tag slow integration tests so they don't run during normal unit tests.

**`integration_test.go`**:
```go
//go:build integration

package main_test
// ... heavy tests
```

Run them with: `go test -tags=integration ./...`

## Interview Questions

**Q: How do you separate platform-specific code in Go?**
**A:** There are two ways:
1.  **File Suffixes**: Create `sys_linux.go` and `sys_windows.go`. The compiler automatically picks the right file.
2.  **Build Tags**: Add `//go:build linux` at the top of the file.
This allows you to define the same function signature (e.g., `func OpenFile(...)`) in multiple files, but only one implementation is compiled into the final binary.

**Q: What happens if you define the same function in `main_linux.go` and `main_windows.go` without build tags?**
**A:** If you are cross-compiling (e.g., `GOOS=linux`), the compiler sees `main_linux.go` and ignores `main_windows.go` due to the suffix convention, so it works fine. However, if you named them `main_1.go` and `main_2.go` (without suffixes) and compiled, you would get a "redeclaration error" because both files would be included in the build.

**Q: Can you combine multiple build tags?**
**A:** Yes. You can use complex boolean logic: `//go:build linux || (darwin && amd64)`.
