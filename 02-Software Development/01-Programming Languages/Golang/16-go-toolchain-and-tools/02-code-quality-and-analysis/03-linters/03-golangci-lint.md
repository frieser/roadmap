# golangci-lint

## Summary
`golangci-lint` is a **linter aggregator**. Instead of installing and running 10 separate tools (vet, staticcheck, revive, errcheck, gosec, etc.) one by one, `golangci-lint` runs them all in parallel, shares the parsing overhead (loading the AST once), and presents a unified output. It is the industry standard for CI/CD pipelines.

## Detailed Explanation

### Architecture
It is written in Go and designed for speed. By reusing the AST (Abstract Syntax Tree) across multiple linters, it drastically reduces execution time compared to shell scripts running tools sequentially.

### Configuration (`.golangci.yml`)
You control everything via a YAML config file in the project root.
```yaml
linters:
  enable:
    - govulncheck
    - staticcheck
    - revive
    - errcheck
    - gosec
  disable:
    - errname

linters-settings:
  errcheck:
    check-type-assertions: true
```

### Popular Included Linters
*   **errcheck**: Ensures you check returned errors.
*   **gosec**: Security scanner.
*   **gocyclo**: Checks cyclomatic complexity (function complexity).
*   **govet**: The standard vet tool.

## Interview Questions

**Q: Why is `golangci-lint` faster than running tools individually?**
**A:** Parsing Go code and type-checking it is expensive (IO and CPU). Individual tools each have to parse the code from scratch. `golangci-lint` parses the code *once* into memory and then feeds that data structures to all enabled linters, saving huge amounts of duplicated work.

**Q: How do you run `golangci-lint` in GitHub Actions?**
**A:** There is an official action `golangci/golangci-lint-action`. It is recommended over manually installing the binary because the Action handles caching (of the Go build cache and linter cache) automatically, making CI runs much faster.

**Q: What is the "fast" mode?**
**A:** `golangci-lint` used to have a fast mode that only ran linters that didn't require type checking. However, almost all useful modern linters require type information to be accurate, so "fast mode" is less relevant today. The tool is fast by design due to parallelism and caching.
