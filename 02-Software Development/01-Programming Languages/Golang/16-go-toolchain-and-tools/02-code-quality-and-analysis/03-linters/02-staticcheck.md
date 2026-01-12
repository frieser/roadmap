# Staticcheck

## Summary
Staticcheck is often described as "**go vet on steroids**". It is an advanced static analysis tool that detects bugs, performance issues, and simplifications that `go vet` misses. Unlike style linters (Revive), Staticcheck focuses on code correctness and efficiency. It is highly respected in the Go community and is used by the Go team itself.

## Detailed Explanation

### What it detects
1.  **Bugs**: Infinite recursion, nil pointer dereferences, impossible type assertions.
2.  **Simplifications**: Suggests replacing complex code with newer/simpler standard library features (e.g., "Use `strings.ReplaceAll` instead of `strings.Replace(..., -1)`").
3.  **Dead Code**: Finds unused constants, variables, and functions.
4.  **Performance**: Suggests pre-allocating slices or using more efficient API calls.

### Usage
```bash
go install honnef.co/go/tools/cmd/staticcheck@latest
staticcheck ./...
```

## Interview Questions

**Q: What is the main difference between Staticcheck and a linter like Revive?**
**A:** Revive complains about *style* (e.g., "variable name should be camelCase", "missing comment"). Staticcheck complains about *logic and quality* (e.g., "this loop will never terminate", "you are copying a large struct by value", "this regex is compiled inside a loop").

**Q: Does Staticcheck support ignoring checks?**
**A:** Yes, via line-based linter directives.
```go
//lint:ignore SA1000 We explicitly want to allow this regex pattern
regexp.MustCompile(...)
```
This granular control allows you to bypass specific checks for valid edge cases without disabling the check globally.

**Q: Why is Staticcheck considered "rigorous"?**
**A:** It performs deep analysis, including data flow analysis, to understand how values propagate through your program. This allows it to find subtle bugs that simple pattern-matching linters cannot detect.
