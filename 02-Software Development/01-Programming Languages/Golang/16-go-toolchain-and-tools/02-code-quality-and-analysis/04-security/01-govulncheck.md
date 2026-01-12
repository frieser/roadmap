# Go Vulncheck

## Summary
`govulncheck` (Go Vulnerability Check) is the official security scanner from the Go team. It checks your code and dependencies against the Go Vulnerability Database. Unlike generic SCA (Software Composition Analysis) tools that just check `go.mod` versions, `govulncheck` uses **call graph analysis** to see if your code *actually calls* the vulnerable function.

## Detailed Explanation

### The Noise Problem
Traditional scanners (like Dependabot or Snyk) often flag a library as "vulnerable" just because you use version X. But if the vulnerability is in function `Foo()`, and your code only calls `Bar()`, you aren't actually affected. This causes "alert fatigue".

### The Solution: Call Graph Analysis
`govulncheck` analyzes your source code (AST) to trace function calls.
1.  Downloads the vulnerability database.
2.  Checks which modules in `go.mod` have known CVEs.
3.  **Crucially**: Checks if your code imports the package AND calls the specific vulnerable function.
4.  Reports only if you are actually reachable/exploitable.

### Usage
```bash
go install golang.org/x/vuln/cmd/govulncheck@latest
govulncheck ./...
```

## Interview Questions

**Q: What is the main advantage of `govulncheck` over generic scanners?**
**A:** Precision. It drastically reduces false positives by verifying **reachability**. It only alerts you if your application actually executes the vulnerable code path, allowing you to prioritize fixes for real threats rather than upgrading libraries that you use safely.

**Q: Where does `govulncheck` get its data?**
**A:** It queries the **Go Vulnerability Database** (https://pkg.go.dev/vuln/), which aggregates data from CVEs, GHSAs (GitHub Security Advisories), and direct reports from Go package maintainers.

**Q: Does `govulncheck` require source code access?**
**A:** Yes. Since it builds a call graph, it needs to parse your source code (or the binary with symbol information) to determine reachability. It cannot work effectively on stripped binaries or without access to the build context.
