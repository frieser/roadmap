# Go Embed

## Summary
The `embed` package (introduced in Go 1.16) allows you to compile static files (HTML, CSS, SQL migrations, images) directly into the Go binary. This creates a single, self-contained executable that is easy to deploy, eliminating the "missing assets" problem at runtime. It uses the special `//go:embed` directive.

## Detailed Explanation

### The `//go:embed` Directive
This is a compiler directive (not a comment) that must immediately precede the variable declaration.

### Embedding Modes

#### 1. Embed as String
Good for small text files (SQL queries, version files).
```go
import _ "embed"

//go:embed version.txt
var version string

func main() {
    fmt.Println("Version:", version)
}
```

#### 2. Embed as []byte
Good for binary files (images, small implementations).
```go
//go:embed logo.png
var logo []byte
```

#### 3. Embed as `embed.FS` (File System)
Best for embedding multiple files or entire directory trees (web assets, migrations). It implements the `fs.FS` interface.
```go
import "embed"

//go:embed static/*
var content embed.FS

func main() {
    // Read specific file from the virtual FS
    data, _ := content.ReadFile("static/index.html")
    
    // Serve with HTTP
    http.Handle("/", http.FileServer(http.FS(content)))
}
```

### Rules & Constraints
*   The variable must be at the **package level** (global), not inside a function.
*   You cannot embed files outside the module root (no `../shadow_passwords`).
*   Pattern matching is supported (globbing): `*.html`.

## Interview Questions

**Q: Can you change the content of an embedded file at runtime?**
**A:** No. Embedded files are read-only and compiled into the binary. To change the content, you must modify the source file and recompile the application. This ensures immutability and reproducibility of the deployment artifact.

**Q: Why does `//go:embed` require the `embed` package to be imported even if you use `string` or `[]byte`?**
**A:** For `embed.FS`, you obviously need the type. For `string` or `[]byte`, you technically don't use a type from the package, but you must import it (often as `import _ "embed"`) to signal the compiler to process the `//go:embed` directives. Without the import, the directive is treated as a regular comment and ignored.

**Q: How does `embed` simplify Docker deployments?**
**A:** Before `embed`, a Dockerfile had to carefully copy the binary *and* the `static/` folder, `templates/` folder, etc., into the image. With `embed`, all those assets are inside the binary. The Dockerfile typically just needs `COPY myapp /myapp`, reducing layer complexity and the risk of missing files.
