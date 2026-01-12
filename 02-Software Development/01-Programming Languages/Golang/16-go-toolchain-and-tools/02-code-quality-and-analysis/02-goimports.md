# Go Imports

## Summary
`goimports` is a tool that updates your Go import lines. It acts as a superset of `gofmt`: it formats the code (fixing indentation and spacing) **AND** adds missing imports / removes unused imports automatically. It is the de-facto standard formatter used in most Go IDEs (VS Code, GoLand) on save.

## Detailed Explanation

### Functionality
1.  **Add Imports**: If you type `json.Marshal`, `goimports` sees the `json` package usage and automatically adds `import "encoding/json"` to the file header.
2.  **Remove Imports**: If you delete the code using `json`, it removes the import line to keep the file clean.
3.  **Sort Imports**: It groups standard library imports separately from third-party imports and sorts them alphabetically.
4.  **Format Code**: It applies all the rules of `gofmt`.

### Usage
```bash
# Install
go install golang.org/x/tools/cmd/goimports@latest

# Run on file (overwrite)
goimports -w main.go
```

## Interview Questions

**Q: Does `goimports` resolve package conflicts?**
**A:** Generally yes, by guessing the most likely package. However, if multiple packages have the same name (e.g., `text/template` and `html/template`), it might pick the wrong one or fail to choose. In such cases, you must add the import manually once, and `goimports` will respect it.

**Q: Why is `goimports` not part of the standard Go distribution?**
**A:** While `gofmt` is part of the core, `goimports` lives in `golang.org/x/tools` (the extended repository). This allows it to evolve faster and depend on larger analysis libraries that the core Go team wants to keep out of the main compiler distribution to keep the binary size small.

**Q: How does `goimports` know where to find packages?**
**A:** It scans your `$GOPATH` and module cache (`go.mod` dependencies) to build an index of available packages and their export symbols.
