# Go Fmt

## Summary
`go fmt` is a command that formats Go source code according to the standard style. It is a wrapper around the `gofmt` tool. It ensures that all Go code looks the same regardless of who wrote it, eliminating debates about coding style (tabs vs spaces, brace position, etc.) and simplifying code reviews.

## Detailed Explanation

### Philosophy
"Gofmt's style is no one's favorite, yet gofmt is everyone's favorite."
Go enforces a single standard format:
*   Tabs for indentation (not spaces).
*   Opening braces on the same line.
*   Strict spacing around operators.

### Usage
```bash
# Format current directory
go fmt .

# Format recursive
go fmt ./...
```

### Underlying Tool: `gofmt`
`go fmt` runs `gofmt -l -w`. You can use `gofmt` directly for more options:
*   `gofmt -w file.go`: Write result to file (overwrite).
*   `gofmt -d file.go`: Display diffs instead of rewriting.
*   `gofmt -r 'pattern -> replacement'`: Rewrite code (simple refactoring).
    ```bash
    gofmt -r 'a[b:len(a)] -> a[b:]' -w main.go
    ```

## Interview Questions

**Q: Does `go fmt` check for syntax errors?**
**A:** Yes. Since it must parse the code into an AST (Abstract Syntax Tree) to format it, it will report syntax errors if the code is invalid. However, it does not check for compilation errors (like type mismatches).

**Q: Why does Go use tabs instead of spaces by default?**
**A:** The Go team decided on tabs for indentation so that developers can set their own display width (2, 4, or 8 spaces) in their editors without changing the source file. It reduces file size and avoids "off-by-one" space errors.

**Q: Is running `go fmt` required to compile the code?**
**A:** No. The Go compiler accepts unformatted code (as long as it is syntactically valid). However, nearly all Go projects enforce `go fmt` via CI/CD pipelines (often using `go fmt ./...` or linters) to maintain codebase consistency.
