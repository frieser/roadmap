# Go Vet

## Summary
`go vet` is the standard static analysis tool included with the Go toolchain. It inspects source code for constructs that look suspicious—code that compiles successfully but is likely a bug (e.g., unreachable code, malformed format strings, or locking a mutex by value). It uses heuristics rather than type checking (which the compiler handles) to find logical errors.

## Detailed Explanation

### How it works
`go vet` runs a set of analyzers on your packages.
```bash
go vet ./...
```
It is automatically run by `go test`, so you get these checks for free when running tests unless you explicitly disable them.

### Common Checks
1.  **printf**: Checks if format strings match arguments.
    ```go
    fmt.Printf("%d", "string") // Vet error: Arg is string, want int
    ```
2.  **structtag**: Checks if struct tags conform to the standard format.
    ```go
    type User struct {
        Name string `json: "name"` // Vet error: space not allowed
    }
    ```
3.  **copylocks**: Checks if you are passing a Lock by value (which copies the internal state, rendering the lock useless).
    ```go
    func do(mu sync.Mutex) {} // Vet error: passing Mutex by value
    ```
4.  **unreachable**: Checks for code that can never be executed.

## Interview Questions

**Q: Does `go vet` catch all bugs?**
**A:** No. `go vet` is conservative by design. It only reports issues where it has high confidence that the code is incorrect. It deliberately avoids "style" or "opinionated" checks (that's what linters like `staticcheck` or `revive` are for) to ensure it can be run by default in `go test` without annoying false positives.

**Q: Why does `go vet` complain if I pass a `sync.Mutex` by value?**
**A:** A `sync.Mutex` contains internal state (flags and semaphores). If you copy it (pass by value), the copy has its own separate state. Locking the copy does not lock the original, destroying the mutual exclusion guarantee. `go vet` detects this via the `copylocks` analyzer.

**Q: Can you write custom analyzers for `go vet`?**
**A:** Yes. The `go/analysis` package allows developers to write their own analysis passes. Tools like `staticcheck` and `golangci-lint` are essentially collections of such analyzers that go beyond the default set included in the standard `go vet` command.
