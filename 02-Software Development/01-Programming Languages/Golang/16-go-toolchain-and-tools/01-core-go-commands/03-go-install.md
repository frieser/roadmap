# Go Install

## Summary
`go install` is used to compile and install Go packages. For executables (commands), it builds the binary and moves it to the `$GOBIN` directory (defaulting to `$GOPATH/bin` or `$HOME/go/bin`). Since Go 1.16, it is also the standard way to install versioned tools globally without affecting the current project's `go.mod`.

## Detailed Explanation

### Installing Local Packages
When run inside a module:
```bash
go install .
```
This builds the current package and installs the binary to `$GOBIN`. This is useful for installing your own tools locally.

### Installing Remote Tools (Global)
You can install tools from remote repositories using the `@version` syntax.
```bash
go install golang.org/x/tools/cmd/godoc@latest
go install github.com/air-verse/air@v1.44.0
```
This downloads the source, compiles it, and places the binary in your global bin path. It ignores the `go.mod` file in the current directory.

### Caching
`go install` caches compiled packages in the Go build cache (`go env GOCACHE`). Subsequent installs are instant if the source hasn't changed.

## Interview Questions

**Q: Where does `go install` put the binary?**
**A:** It places the binary in the directory specified by the `GOBIN` environment variable. If `GOBIN` is not set, it defaults to `$GOPATH/bin` (usually `$HOME/go/bin`). You must ensure this directory is in your system `$PATH` to run the installed commands directly.

**Q: Can `go install` be used to update dependencies in `go.mod`?**
**A:** No. Before Go 1.16, `go get` was used for both installing tools and adding dependencies. Now, the roles are split: use `go get` to add/update dependencies in `go.mod`, and use `go install` to build and install executable commands.

**Q: What happens if you run `go install` on a non-main package (a library)?**
**A:** It compiles the package and caches the result (the object files) in the generic build cache (`$GOCACHE`), speeding up future builds that depend on it. However, since there is no `main` function, no executable binary is created or moved to `bin/`.
