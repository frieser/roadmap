#Golang
---
---

## Summary

The `go` command is Go's unified toolchain for building, testing, managing dependencies, and maintaining code. It replaces the need for separate build systems like Make or Maven. Key commands include `go build`, `go run`, `go test`, `go mod`, `go fmt`, and `go vet`. Understanding these commands is essential for effective Go development.

## Detailed Explanation

### **Command Overview**

```bash
go <command> [arguments]

# Get help
go help              # List all commands
go help build        # Detailed help for a command
go help modules      # Help on topic
```

### **Core Commands**

#### go run - Compile and Execute

```bash
# Run a single file
go run main.go

# Run multiple files
go run main.go utils.go

# Run all Go files in directory
go run .

# Pass arguments to the program
go run main.go arg1 arg2
```

Behavior:
- Compiles to a temporary binary
- Executes immediately
- Deletes binary after execution
- For development only—use `go build` for production

#### go build - Compile Packages

```bash
# Build current package
go build

# Build with output name
go build -o myapp

# Build specific file
go build main.go

# Build for different OS/architecture
GOOS=linux GOARCH=amd64 go build -o myapp-linux

# Build with optimizations (strip debug info)
go build -ldflags="-s -w" -o myapp
```

Common flags:

| Flag | Purpose |
| --- | --- |
| `-o name` | Output file name |
| `-v` | Verbose (print package names) |
| `-race` | Enable race detector |
| `-ldflags` | Pass flags to the linker |
| `-tags` | Build tags to consider |
| `-trimpath` | Remove file paths from binary |

#### go install - Compile and Install

```bash
# Install current package to $GOPATH/bin
go install

# Install a remote package
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest

# Install specific version
go install github.com/cosmtrek/air@v1.49.0
```

Difference from `go build`:
- `go build` creates binary in current directory
- `go install` creates binary in `$GOBIN` (usually `$GOPATH/bin`)

### **Module Commands (go mod)**

```bash
# Initialize a new module
go mod init github.com/username/project

# Add missing and remove unused dependencies
go mod tidy

# Download dependencies to local cache
go mod download

# Verify dependencies haven't been modified
go mod verify

# Print module dependency graph
go mod graph

# Create a vendor directory
go mod vendor

# Edit go.mod programmatically
go mod edit -require=github.com/gin-gonic/gin@v1.9.1
go mod edit -go=1.24
```

Example workflow:

```bash
# Start new project
mkdir myproject && cd myproject
go mod init github.com/user/myproject

# Write code with imports
cat > main.go << 'EOF'
package main

import (
    "fmt"
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()
    r.GET("/", func(c *gin.Context) {
        c.String(200, "Hello")
    })
    fmt.Println("Starting server...")
    r.Run()
}
EOF

# Fetch dependencies
go mod tidy

# Check go.mod and go.sum
cat go.mod
```

### **Testing Commands (go test)**

```bash
# Run tests in current package
go test

# Run tests with verbose output
go test -v

# Run specific test
go test -run TestFunctionName

# Run tests in all packages
go test ./...

# Run with coverage
go test -cover
go test -coverprofile=coverage.out

# Run benchmarks
go test -bench=.
go test -bench=BenchmarkName

# Run with race detector
go test -race

# Set timeout
go test -timeout 30s
```

Common flags:

| Flag | Purpose |
| --- | --- |
| `-v` | Verbose output |
| `-run regex` | Run only matching tests |
| `-bench regex` | Run matching benchmarks |
| `-cover` | Enable coverage analysis |
| `-coverprofile file` | Write coverage to file |
| `-race` | Enable race detector |
| `-count n` | Run tests n times |
| `-parallel n` | Max parallel tests |
| `-short` | Skip long-running tests |

#### Coverage Report

```bash
# Generate coverage profile
go test -coverprofile=coverage.out ./...

# View in terminal
go tool cover -func=coverage.out

# View in browser (HTML)
go tool cover -html=coverage.out
```

### **Code Quality Commands**

#### go fmt - Format Code

```bash
# Format a file
go fmt main.go

# Format current package
go fmt .

# Format all packages
go fmt ./...

# Check if formatting needed (useful in CI)
gofmt -l .  # List files that need formatting
```

#### go vet - Static Analysis

```bash
# Analyze current package
go vet

# Analyze all packages
go vet ./...

# Specific checks
go vet -printf=true ./...
```

Detects issues like:
- Unreachable code
- Printf format string mismatches
- Suspicious constructs
- Incorrect use of sync/atomic

#### goimports - Format + Fix Imports

```bash
# Install
go install golang.org/x/tools/cmd/goimports@latest

# Run
goimports -w .  # -w writes changes
```

### **Environment Commands**

```bash
# Print all environment variables
go env

# Print specific variable
go env GOPATH
go env GOOS

# Set environment variable permanently
go env -w GOBIN=$HOME/bin

# Unset environment variable
go env -u GOBIN

# Print in JSON format
go env -json
```

Key environment variables:

| Variable | Description |
| --- | --- |
| `GOOS` | Target operating system |
| `GOARCH` | Target architecture |
| `GOPATH` | Workspace location |
| `GOROOT` | Go installation location |
| `GOPROXY` | Module proxy URL |
| `GOPRIVATE` | Private module patterns |

### **Documentation Commands**

```bash
# View package documentation
go doc fmt
go doc fmt.Println

# View source code
go doc -src fmt.Println

# View all exported symbols
go doc -all fmt

# Start local documentation server
go doc -http=:6060  # Then open http://localhost:6060
```

### **Other Useful Commands**

#### go get - Add/Update Dependencies

```bash
# Add a dependency
go get github.com/gin-gonic/gin

# Add specific version
go get github.com/gin-gonic/gin@v1.9.1

# Update to latest
go get -u github.com/gin-gonic/gin

# Update all dependencies
go get -u ./...
```

#### go clean - Remove Build Artifacts

```bash
# Clean current package
go clean

# Clean and remove cached build files
go clean -cache

# Clean and remove test cache
go clean -testcache

# Clean and remove module cache
go clean -modcache
```

#### go generate - Code Generation

```bash
# Run generate directives
go generate ./...
```

Usage in code:

```go
//go:generate stringer -type=Status
type Status int

const (
    Pending Status = iota
    Running
    Complete
)
```

### **Cross-Compilation**

```bash
# Build for Linux from any OS
GOOS=linux GOARCH=amd64 go build -o app-linux

# Build for Windows
GOOS=windows GOARCH=amd64 go build -o app.exe

# Build for macOS ARM
GOOS=darwin GOARCH=arm64 go build -o app-mac

# Build for multiple platforms
for os in linux darwin windows; do
    for arch in amd64 arm64; do
        GOOS=$os GOARCH=$arch go build -o "app-$os-$arch"
    done
done
```

Supported platforms:

| GOOS | GOARCH |
| --- | --- |
| linux | amd64, arm64, arm, 386 |
| darwin | amd64, arm64 |
| windows | amd64, arm64, 386 |
| freebsd | amd64, arm64 |

### **Quick Reference Cheat Sheet**

```bash
# Development workflow
go mod init example.com/project   # Initialize module
go mod tidy                       # Sync dependencies
go run .                          # Run program
go test -v ./...                  # Run all tests
go build -o app                   # Build binary

# Code quality
go fmt ./...                      # Format code
go vet ./...                      # Static analysis
golangci-lint run                 # Comprehensive linting

# Dependencies
go get package@version            # Add dependency
go mod download                   # Download all dependencies
go mod vendor                     # Vendor dependencies

# Inspection
go doc package.Symbol             # View documentation
go env                            # View environment
go list -m all                    # List all modules
```

## Interview Questions

**Q: What is the difference between `go run`, `go build`, and `go install`?**
**A:** `go run` compiles and executes immediately using a temporary binary—ideal for development. `go build` compiles and creates a binary in the current directory. `go install` compiles and places the binary in `$GOBIN` (typically `$GOPATH/bin`), making it available system-wide via PATH.

**Q: How do you cross-compile Go programs?**
**A:** Set the `GOOS` and `GOARCH` environment variables before building. For example, `GOOS=linux GOARCH=amd64 go build` creates a Linux binary from any OS. Go supports cross-compilation natively without additional toolchains (unless using CGO).

**Q: What does `go mod tidy` do?**
**A:** `go mod tidy` analyzes your source code, adds any missing module dependencies to `go.mod`, removes unused dependencies, and updates `go.sum` with the correct checksums. It ensures your module files accurately reflect your code's actual dependencies.

**Q: How do you check test coverage in Go?**
**A:** Run `go test -cover` for a summary percentage, or `go test -coverprofile=coverage.out` to generate a detailed profile. View the profile with `go tool cover -func=coverage.out` (text) or `go tool cover -html=coverage.out` (interactive HTML report showing uncovered lines).
