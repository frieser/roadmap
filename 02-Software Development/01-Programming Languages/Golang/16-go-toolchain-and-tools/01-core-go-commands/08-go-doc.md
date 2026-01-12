# Go Doc

## Summary
`go doc` is a command-line tool that displays documentation for Go packages and symbols (functions, types, constants) directly in the terminal. It parses the comments in the source code (specifically comments starting with the symbol name or Package) and presents them in a readable format, similar to Javadoc or Pydoc.

## Detailed Explanation

### Usage
```bash
# Show documentation for a package
go doc json

# Show documentation for a specific symbol
go doc json.Marshal

# Show documentation for a method
go doc http.Client.Do

# Show private symbols too
go doc -u json
```

### Doc Comments
Go treats comments appearing immediately before a declaration as documentation.
```go
// Add returns the sum of two integers.
func Add(a, b int) int { ... }
```
Running `go doc Add` will display this comment.

## Interview Questions

**Q: What is the difference between `go doc` and `godoc`?**
**A:** `go doc` is the standard command-line tool included in the Go distribution for quick lookups. `godoc` (now a separate module `golang.org/x/tools/cmd/godoc`) creates a local web server (http://localhost:6060) that hosts the full documentation of the standard library and your project in a browser-friendly format.

**Q: How do you format code blocks in Go documentation comments?**
**A:** Indent the text with a tab or spaces. Go doc (and the web viewer) treats indented lines as pre-formatted code blocks, preserving spacing and ignoring markdown-like formatting.
