# Go Generate

## Summary
`go generate` is a standard Go command used to automate the running of code generation tools. It scans Go source files for magic comments (`//go:generate ...`) and executes the specified commands. This allows you to generate code (mocks, string methods, protobufs, asset bundles) as part of your development workflow, rather than during the build process.

## Detailed Explanation

### How it Works
`go generate` is **not** part of `go build`. It must be run explicitly.
1.  You add a directive in your code:
    ```go
    //go:generate command arguments...
    ```
2.  You run `go generate ./...` in your terminal.
3.  The tool scans files, finds directives, and runs the commands.

### Common Use Cases

#### 1. Generating `String()` methods (`stringer`)
The `stringer` tool automates the creation of `String()` methods for integer constants (enums).

```go
package painkiller

//go:generate stringer -type=Pill
type Pill int

const (
	Placebo Pill = iota
	Aspirin
	Ibuprofen
	Paracetamol
)
```
Running `go generate` creates a `pill_string.go` file with an efficient `func (i Pill) String() string` implementation.

#### 2. Generating Mocks (`mockgen`)
```go
//go:generate mockgen -destination=mocks/mock_db.go -package=mocks . DB
type DB interface { ... }
```

### Best Practices
*   **Commit Generated Code**: Always commit the generated files to Git. This ensures that the project can be built by anyone (including CI) without needing to install the specific generator tools.
*   **No Build Dependencies**: Since generated code is checked in, `go build` works out of the box. `go generate` is only for the developer modifying the source.
*   **Recursive**: Use `go generate ./...` to run all generators in the project.

## Interview Questions

**Q: Is `go generate` run automatically by `go build`?**
**A:** No. `go generate` is intended to be run by the developer *authoring* the code, not by the person *building* the code. You run it to update the generated files, and then you commit those files. This ensures that the build process remains simple and doesn't require external tools (like `stringer` or `protoc`) to be present on the build machine.

**Q: What is the significance of the space in `//go:generate`?**
**A:** There must be **no space** between `//` and `go:generate`.
*   Correct: `//go:generate ...`
*   Incorrect: `// go:generate ...` (treated as a regular comment)

**Q: Can `go generate` run arbitrary shell scripts?**
**A:** Yes. The command after `//go:generate` is executed by the shell. You can run `bash scripts/gen.sh`, `python codegen.py`, or any binary in your `$PATH`.
